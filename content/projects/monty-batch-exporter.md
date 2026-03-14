---
title: "Monty Batch Exporter"
date: 2026-03-12
draft: false
tech_stack: ["Python", "Blender API", "bpy"]
hero_video_id: ""
thumbnail_url: "/images/thumbnails/MontyBatchExporter_Thumb.png"
summary: "A Blender 4.2+ extension that batch-exports objects, collections, or scenes as individual files — with configurable naming, filtering, preset support, and CLI automation."
project_type: "tool"
tags: ["pipeline", "python", "automation", "blender", "export"]
---

## Overview

Monty Batch Exporter is a Blender extension that wraps every
native exporter (FBX, glTF, OBJ, USD, Alembic, STL, PLY,
Collada) into a single batch-export operation. Artists pick a
scope — selected objects, collections, scenes, or all mesh
objects — configure naming rules, and export hundreds of files
with one click.

The tool ships as a standard Blender extension package and
includes a command-line interface for headless pipeline
automation.

{{< vimeo id="1173607042" title="MontyBatchExporter_Demo" >}}

## The Problem

In any 3D production pipeline, assets eventually need to leave
the DCC application. A character artist finishes a model and
exports it as FBX for the game engine; an environment artist
exports dozens of props for a level; a tech artist ships LOD
chains with matching textures.

Blender's built-in export operators handle one file at a time.
For a scene with 50 props, that means:

- Manually selecting each object
- Opening the export dialog
- Typing a correct filename
- Choosing the right directory
- Clicking export
- Repeating 49 more times

This process is slow, error-prone, and impossible to
standardise across a team. Filenames drift, objects end up
in wrong folders, and an artist's afternoon disappears into
repetitive clicking.

**The goal:** reduce a multi-hour manual export session to a
single operation — configurable, repeatable, and scriptable.

## Technical Implementation

### Architecture

The codebase is split into two layers with a deliberate
dependency boundary:

```
ProjectC/
├── core/           ← Pure Python, zero DCC dependencies
│   ├── naming.py   ← Naming convention engine
│   ├── config.py   ← Export job configuration (dataclass)
│   ├── file_io.py  ← Safe directory creation & validation
│   └── report.py   ← Export result aggregation
├── blender/        ← Blender 4.2+ extension (bpy-dependent)
│   ├── exporter.py ← Native operator wrapper + preset loader
│   ├── operators.py← Main export loop + save/load config
│   ├── ui.py       ← PropertyGroup + menu registration
│   ├── utils.py    ← Selection isolation, gather strategies
│   └── preferences.py
└── cli_utils.py    ← Headless pipeline helpers (config validation, patching)
```

The `core/` package has **zero** imports from `bpy` or any
DCC application. This means:

- Unit tests run instantly with plain `pytest` — no Blender
  process needed
- The same naming, config, and reporting logic can be reused
  in a future Maya or Houdini port
- The core can be developed and validated independently

### Naming Engine

One of the trickiest parts of batch export is generating correct,
consistent filenames. The naming engine assembles filenames from
configurable tokens:

```python
@dataclass
class NamingRule:
    prefix: str = ""
    suffix: str = ""
    separator: str = "_"
    use_object_name: bool = True
    use_collection_name: bool = False
    counter_digits: int = 0
```

The resolution order is:
`prefix` → `collection_name` → `object_name` → `suffix` → `counter`,
joined by the separator. Every token passes through a sanitiser
that strips characters unsafe across Windows, macOS, and Linux
filesystems.

A live preview in the UI updates as the artist types, so they
see the exact filename pattern before committing to an export:

<img src="/images/BatchExport_NamingPreviewScreenshot.png"
     alt="Naming preview UI" style="width:100%; border-radius:8px; margin: 1rem 0;">

### Selection Isolation

Blender's native exporters rely on `bpy.context.selected_objects`
to determine what to export. Batch export needs to temporarily
manipulate selection without disrupting the artist's scene state.

The `isolate_selection` context manager snapshots the current
selection, swaps in only the target objects, yields control to
the exporter, then restores the original state — even if the
export fails:

```python
@contextlib.contextmanager
def isolate_selection(objects):
    prev_active = bpy.context.view_layer.objects.active
    prev_selected = [
        obj for obj in bpy.context.view_layer.objects
        if obj.select_get()
    ]
    try:
        bpy.ops.object.select_all(action="DESELECT")
        for obj in objects:
            obj.select_set(True)
        bpy.context.view_layer.objects.active = objects[0]
        yield
    finally:
        # Always restore, even on error
        bpy.ops.object.select_all(action="DESELECT")
        for obj in prev_selected:
            obj.select_set(True)
        bpy.context.view_layer.objects.active = prev_active
```

The same pattern applies to `export_at_origin`, which temporarily
moves objects to world origin `(0, 0, 0)` during export and
restores their positions afterwards — critical for game engines
that expect assets centred at the origin.

### Cross-Scene Export

A non-obvious challenge: Blender's export operators only see
objects in the **active scene**. If a `.blend` file contains
props spread across multiple scenes, a naive "export all"
misses everything outside the current one.

The solution switches `bpy.context.window.scene` per item
during the export loop, ensuring every scene's objects are
reachable, then restores the original scene at the end:

```python
original_scene = bpy.context.window.scene
try:
    for item in items:
        target = bpy.data.scenes.get(item.scene_name)
        if target and target != bpy.context.window.scene:
            bpy.context.window.scene = target

        with isolate_selection(item.objects), \
             export_at_origin(item.objects, enabled=config.export_at_origin):
            export_single(filepath=str(out_path), ...)
finally:
    bpy.context.window.scene = original_scene
```

### Native Preset Integration

Artists often have carefully tuned FBX or glTF export presets
(axis conventions, scale, mesh options). Rather than duplicating
those settings, the tool discovers and loads Blender's native
operator presets at runtime:

```python
def load_preset_kwargs(preset_path: str) -> dict:
    kwargs = {}
    for line in Path(preset_path).read_text().splitlines():
        if not line.strip().startswith("op."):
            continue
        prop, _, value_str = line.split("op.", 1)[1].partition(" = ")
        if prop not in _PRESET_SKIP:
            kwargs[prop] = ast.literal_eval(value_str)
    return kwargs
```

The parsed keyword arguments are passed directly to the native
export operator, so artists get their exact configured behaviour
without re-entering settings.

### CLI & Config Persistence

For pipeline automation, the tool exposes its operators to
Blender's `--python-expr` flag, so any `.blend` file can be
batch-exported headlessly with a single command:

```bash
blender "path/to/scene.blend" --background --python-expr \
    "import bpy; bpy.ops.batch_exporter.load_config('EXEC_DEFAULT', filepath=r'path/to/batch_export_config.json'); bpy.ops.batch_exporter.export('EXEC_DEFAULT')"
```

The config JSON is produced by a "Save Config" button in the UI,
capturing every setting (scope, format, naming rules, filters,
advanced options). This enables:

- **CI/CD integration** — batch exports triggered on asset commits
- **Cross-artist consistency** — shared config files ensure the
  same naming and structure across a team
- **Reproducibility** — re-run an identical export months later

### Key Features

| Feature | Description |
|---|---|
| **4 export scopes** | Selected objects, collections, scenes, all mesh objects |
| **8 formats** | FBX, glTF, OBJ, USD, Alembic, STL, PLY, Collada |
| **Naming engine** | Prefix, suffix, separator, object/collection tokens, counters |
| **Subdirectory grouping** | Auto-organise output by prefix or collection |
| **Name filtering** | Export only items matching a prefix/suffix pattern |
| **Native presets** | Load Blender's built-in export presets |
| **Export at origin** | Temporarily zero-out transforms for export |
| **LOD packing** | Group LOD variants into single files |
| **Texture export** | Copy referenced textures alongside exported meshes |
| **Save/Load config** | JSON round-trip for all export settings |
| **CLI mode** | Headless batch export for pipelines |

## Process

{{< collapse summary="Development approach and QA methodology" >}}

The project followed a structured development cycle:

1. **Core-first development** — the naming engine, config model,
   file I/O, and reporting module were built and unit-tested
   before any Blender code was written. This validated the
   design with fast `pytest` feedback loops.

2. **Blender integration** — operators, UI, and utility helpers
   were built on top of the tested core, using Blender's
   `bpy.types.Operator` and `PropertyGroup` patterns.

3. **QA test plans** — formal JSON-based test plans defined
   expected behaviour for each feature. Three QA rounds
   (v1 → v2 → v3) identified and resolved bugs including:
   - Cross-scene export failures
   - Chained error tracebacks leaking to artists
   - Redundant UI elements

4. **Extension packaging** — adapted to Blender's new extension
   manifest system (`blender_manifest.toml`), converted all
   imports to relative, created a build script that assembles
   the distributable `.zip`.

5. **Git Flow** — feature branch → develop → release-1.0.0 →
   main with `--no-ff` merges, annotated `v1.0.0` tag, pushed
   to GitHub.

{{< /collapse >}}

## Results

{{< alert success >}}
A manual export session of 50+ assets that previously took
30–60 minutes of repetitive clicking now completes in a single
operation — typically under 30 seconds.
{{< /alert >}}

| Metric | Before | After |
|---|---|---|
| Time to export 50 assets | 30–60 min (manual) | < 30 sec |
| Naming consistency | Ad-hoc, error-prone | Enforced by naming engine |
| Pipeline automation | Not possible | JSON config + CLI |
| Cross-scene export | Manual scene switching | Automatic |
| Preset reuse | Re-enter settings each time | Native preset loading |

### Codebase

| Module | Lines | Purpose |
|---|---|---|
| `core/` | 525 | DCC-agnostic library (naming, config, I/O, reporting) |
| `blender/` | 1,762 | Blender extension (operators, UI, exporter, utils) |
| `cli_utils.py` + build | 420 | Pipeline helpers and packaging script |
| `tests/` | 388 | 32 unit tests (pytest, no Blender needed) |
| **Total** | **3,095** | **17 Python files** |

## Get It

Monty Batch Exporter is available on three platforms:

{{< button link="#" text="Blender Extensions Platform" target="_blank" >}}
<!-- TODO: Replace # with extensions.blender.org URL once approved -->

{{< button link="https://github.com/MontyAnim/ProjectC" text="GitHub Repository" target="_blank" >}}

{{< button link="#" text="Gumroad" target="_blank" >}}
<!-- TODO: Replace # with Gumroad product URL -->

## Retrospective

### What I Learned

- **Blender's extension system** (introduced in 4.2) requires
  careful attention to relative imports and manifest fields —
  the packaging model is fundamentally different from the old
  add-on system.

- **Context managers** (`isolate_selection`, `export_at_origin`)
  proved to be the cleanest pattern for temporarily mutating
  Blender state. The `try/finally` guarantee is essential when
  an export might fail mid-operation.

- **Separating pure logic from DCC code** paid off immediately:
  the naming engine has 12 unit tests that run in milliseconds,
  catching regressions long before launching Blender.

### What I Would Do Differently

- **Start with the extension manifest earlier.** I initially
  used an absolute-path symlink for development convenience,
  which broke on every other machine. Planning the packaging
  format upfront would have avoided the late refactor.

- **Add integration tests inside Blender.** The current test
  suite covers the core library thoroughly, but the Blender
  operators are only tested manually via QA plans. A
  `blender --background --python run_tests.py` harness would
  close that gap.

- **Expose more granular progress feedback.** The current
  progress indicator works, but for very large exports
  (hundreds of files), a per-file progress bar in Blender's
  status bar would improve the artist experience.
