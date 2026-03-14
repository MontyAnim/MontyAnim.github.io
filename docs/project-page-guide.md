# Project Page Authoring Guide

How to create and maintain project pages for this Hugo portfolio.

---

## Table of Contents

1. [Creating a New Project Page](#1-creating-a-new-project-page)
2. [Front Matter Reference](#2-front-matter-reference)
3. [Thumbnail & Hero Images](#3-thumbnail--hero-images)
4. [Placeholder Images (No Real Asset Yet)](#4-placeholder-images-no-real-asset-yet)
5. [Videos — YouTube](#5-videos--youtube)
6. [Videos — Vimeo](#6-videos--vimeo)
7. [Placeholder Videos (No Real Video Yet)](#7-placeholder-videos-no-real-video-yet)
8. [Code Snippets (Inline)](#8-code-snippets-inline)
9. [GitHub Gist Embeds](#9-github-gist-embeds)
10. [GitHub File Embeds](#10-github-file-embeds)
11. [Image Gallery](#11-image-gallery)
12. [WebGL / Interactive Demos](#12-webgl--interactive-demos)
13. [Callout Alerts](#13-callout-alerts)
14. [Buttons](#14-buttons)
15. [Collapsible Sections](#15-collapsible-sections)
16. [Publishing a Draft](#16-publishing-a-draft)
17. [Full Example Project Page](#17-full-example-project-page)

---

## 1. Creating a New Project Page

Run the Hugo archetype command from the repo root:

```powershell
hugo new content projects/your-project-name.md
```

This creates `content/projects/your-project-name.md` pre-filled from the archetype template. The filename becomes the URL slug: `/projects/your-project-name/`.

**Naming conventions:**
- Lowercase, words separated by hyphens
- No spaces, no underscores
- Examples: `stylized-water-shader.md`, `maya-batch-exporter.md`, `ue5-terrain-tool.md`

---

## 2. Front Matter Reference

Every project file starts with a YAML block between `---` markers.

```yaml
---
title: "My Project Title"
date: 2026-03-12
draft: true
tech_stack: ["Python", "Unity", "HLSL"]
hero_video_id: ""
thumbnail_url: "/images/thumbnails/my-project.gif"
summary: "One sentence description shown on the projects list and homepage cards."
project_type: "tool"
tags: ["pipeline", "python", "automation"]
---
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | string | ✅ | Displayed as the page `<h1>` |
| `date` | YYYY-MM-DD | ✅ | Used for ordering (newest first) |
| `draft` | bool | ✅ | `true` = hidden from live site; `false` = published |
| `tech_stack` | list | ✅ | Shown as monospace badges on cards and project page |
| `hero_video_id` | string | — | Vimeo video ID — rendered above content if set |
| `thumbnail_url` | string | ✅ | Path to card thumbnail (stored in `static/images/thumbnails/`) |
| `summary` | string | ✅ | 1–2 sentence description for cards. Keep it under 160 characters |
| `project_type` | string | ✅ | `"tool"` or `"visual"` — controls the filter tabs on the projects page |
| `tags` | list | — | Generates tag pages. Common: `automation`, `python`, `hlsl`, `unity`, `maya`, `pipeline`, `shaders`, `vfx` |

---

## 3. Thumbnail & Hero Images

Thumbnails appear on the project cards (homepage and projects list page).

**Storage location:** `static/images/thumbnails/`

**Supported formats:** `.gif` (preferred for animated previews), `.png`, `.jpg`, `.webp`

**Recommended size:** 800×450 px (16:9 ratio)

**To add a thumbnail:**

1. Copy your image to `static/images/thumbnails/`
2. Set `thumbnail_url` in front matter:

```yaml
thumbnail_url: "/images/thumbnails/my-project.gif"
```

**Images inside the page body:**

```markdown
<!-- Standard Markdown (no caption) -->
![Alt text](/images/thumbnails/my-project.gif)

<!-- Figure shortcode — adds a styled caption -->
{{< figure src="/images/my-project/ui-overview.png" alt="Tool UI overview" caption="The batch exporter main window" >}}

<!-- Centered figure -->
{{< figure src="/images/my-project/shader.png" alt="Shader preview" caption="Water shader in Unity" align="center" >}}
```

---

## 4. Placeholder Images (No Real Asset Yet)

**Option A — placehold.co (text labels, no tracking, works offline-ish):**

```html
<img src="https://placehold.co/800x450/ede4d3/7a6652?text=Screenshot+Coming+Soon"
     alt="Coming soon" style="width:100%; border-radius:8px; margin: 1rem 0;">
```

Palette-matched hex values: `ede4d3` = surface, `7a6652` = secondary text.

**Option B — picsum.photos (random photos):**

```html
<img src="https://picsum.photos/800/450" alt="Placeholder"
     style="width:100%; border-radius:8px; margin: 1rem 0;">
```

**Option C — Local placeholder file** (best for offline work):

1. Drop any 800×450 image into `static/images/thumbnails/placeholder.png`
2. Reference it:

```yaml
thumbnail_url: "/images/thumbnails/placeholder.png"
```

---

## 5. Videos — YouTube

Get the video ID from the URL:
`https://www.youtube.com/watch?v=dQw4w9WgXcQ` → ID is `dQw4w9WgXcQ`

```
{{</* youtube id="dQw4w9WgXcQ" */>}}
```

**Start at a specific timestamp (seconds):**
```
{{</* youtube id="dQw4w9WgXcQ" start="42" */>}}
```

**With accessibility title:**
```
{{</* youtube id="dQw4w9WgXcQ" title="Stylized Water Shader Demo" */>}}
```

Renders in a responsive 16:9 container automatically.

---

## 6. Videos — Vimeo

Get the video ID from the URL:
`https://vimeo.com/123456789` → ID is `123456789`

```
{{</* vimeo id="123456789" */>}}
```

**Autoplay + loop (great for silent shader/VFX demos):**
```
{{</* vimeo id="123456789" autoplay="1" loop="1" muted="1" */>}}
```

**With accessibility title:**
```
{{</* vimeo id="123456789" title="Water Shader Breakdown" */>}}
```

> `autoplay` only works when `muted="1"` — this is a browser security requirement.

**Hero video** — To place a Vimeo video above all written content (in the hero area of the project page), set `hero_video_id` in front matter instead of using the shortcode:

```yaml
hero_video_id: "123456789"
```

---

## 7. Placeholder Videos (No Real Video Yet)

Drop this HTML block where the video will eventually go:

```html
<div style="position: relative; padding-bottom: 56.25%; height: 0; background: var(--surface); border: 2px dashed var(--border); border-radius: 8px; margin: 1.5rem 0;">
  <div style="position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; color: var(--secondary); gap: 0.5rem;">
    <span style="font-size: 3rem;">🎬</span>
    <span style="font-weight: 600; font-family: sans-serif;">Video Coming Soon</span>
    <span style="font-size: 0.875rem; font-family: sans-serif;">Demo recording in progress</span>
  </div>
</div>
```

This matches the site palette and renders at a proper 16:9 aspect ratio.

---

## 8. Code Snippets (Inline)

Use fenced code blocks with a language tag for syntax highlighting:

**Python:**
````markdown
```python
class AssetValidator:
    def check_naming(self, name: str) -> bool:
        return bool(re.match(r'^[A-Z][a-zA-Z0-9_]+$', name))
```
````

**HLSL:**
````markdown
```hlsl
float4 frag(v2f i) : SV_Target {
    float depth = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
    return float4(depth, depth, depth, 1.0);
}
```
````

**C#:**
````markdown
```csharp
[SerializeField] private Material waterMaterial;

void Update() {
    waterMaterial.SetFloat("_Time", Time.time);
}
```
````

**Supported language tags:** `python`, `csharp`, `hlsl`, `glsl`, `json`, `yaml`, `bash`, `powershell`, `javascript`, `html`, `css`

**Inline code** (single backtick) for referencing symbols within prose:

```markdown
The `AssetValidator` class handles naming convention checks.
```

---

## 9. GitHub Gist Embeds

Create a Gist at [gist.github.com](https://gist.github.com). Copy the Gist ID from the URL:
`https://gist.github.com/MontyAnim/abc123def456` → ID is `abc123def456`

**Embed the entire gist:**
```
{{</* gist MontyAnim abc123def456 */>}}
```

**Embed a single file from a multi-file gist:**
```
{{</* gist MontyAnim abc123def456 "validator.py" */>}}
```

> Gists render with GitHub's default styling. Best for sharing larger, shareable standalone files.

---

## 10. GitHub File Embeds

Embeds a file (or line range) from a public GitHub repo with syntax highlighting and a "View on GitHub" link.

**Full file:**
```
{{</* github-code user="MontyAnim" repo="maya-batch-exporter" file="src/validator.py" */>}}
```

**Specific line range:**
```
{{</* github-code user="MontyAnim" repo="maya-batch-exporter" file="src/validator.py" lines="10-35" */>}}
```

**With explicit language:**
```
{{</* github-code user="MontyAnim" repo="water-shader" file="Shaders/Water.hlsl" lang="hlsl" */>}}
```

> Default language is `python`. The file must be in a **public** repository. Powered by emgithub.com.

---

## 11. Image Gallery

Displays multiple images in a responsive row, comma-separated.

```
{{</* gallery images="/images/my-project/step1.png, /images/my-project/step2.png, /images/my-project/result.png" */>}}
```

**Recommended folder structure:**
```
static/
  images/
    my-project-name/
      overview.png
      wireframe.png
      final-result.gif
```

Then reference as `/images/my-project-name/overview.png`.

---

## 12. WebGL / Interactive Demos

For Unity WebGL builds or other browser-based interactive demos.

**Setup:**
1. Export your Unity WebGL build
2. Place the entire exported folder inside `static/demos/`:

```
static/
  demos/
    water-demo/
      index.html
      Build/
        ...
      TemplateData/
        ...
```

**Embed in your project page:**
```
{{</* webgl demo="water-demo" title="Interactive Water Shader Demo" */>}}
```

**Custom height (default: 600px):**
```
{{</* webgl demo="water-demo" title="Water Demo" height="800px" */>}}
```

Renders in a sandboxed `<iframe>`. Unity WebGL requires `Build/` to be alongside `index.html`.

---

## 13. Callout Alerts

For notes, warnings, or outcomes that should stand out.

```
{{</* alert info */>}}
This shader requires Unity 2022.3 LTS or newer.
{{</* /alert */>}}
```

```
{{</* alert warning */>}}
This tool modifies your Maya scene in-place — always work on a duplicate.
{{</* /alert */>}}
```

```
{{</* alert success */>}}
Batch export time reduced from ~4 hours to under 8 minutes.
{{</* /alert */>}}
```

```
{{</* alert error */>}}
Known issue: LOD generation fails on meshes over 500k triangles.
{{</* /alert */>}}
```

**Available types:** `info`, `warning`, `success`, `error`

---

## 14. Buttons

Adds a styled call-to-action link.

```
{{</* button link="https://github.com/MontyAnim/my-tool" text="View Source on GitHub" target="_blank" */>}}
```

```
{{</* button link="/contact/" text="Get in Touch" */>}}
```

| Parameter | Required | Description |
|---|---|---|
| `link` | ✅ | URL — absolute for external, `/path/` for internal |
| `text` | ✅ | Button label |
| `target` | — | Use `"_blank"` for external links |

---

## 15. Collapsible Sections

For technical appendices, full source listings, or long breakdowns that not every reader needs.

```
{{</* collapse summary="Full shader source (HLSL)" */>}}

```hlsl
// Full code here...
```

{{</* /collapse */>}}
```

**Open by default:**
```
{{</* collapse summary="Performance benchmarks" openByDefault=true */>}}
Content visible on page load.
{{</* /collapse */>}}
```

---

## 16. Publishing a Draft

When a project is ready to go live, change the front matter:

```yaml
draft: false
```

Then commit and push. Merge to `develop` for team review, or open a PR to `main` to trigger the GitHub Pages deployment.

**Local preview (shows drafts):**
```powershell
hugo server -D
```

**Local preview (production mode — hides drafts, same as live site):**
```powershell
hugo server
```

---

## 17. Full Example Project Page

```markdown
---
title: "Procedural Terrain Generator"
date: 2026-03-01
draft: false
tech_stack: ["C#", "Unreal Engine", "HLSL"]
hero_video_id: ""
thumbnail_url: "/images/thumbnails/terrain-generator.gif"
summary: "Runtime procedural terrain system for UE5 using GPU compute shaders and layered noise."
project_type: "tool"
tags: ["pipeline", "unreal", "hlsl", "procedural"]
---

## Overview

A runtime terrain generation system built for Unreal Engine 5 that uses compute shaders to generate
layered noise-based terrain meshes, integrated directly into the Level Editor.

## Demo

{{</* youtube id="dQw4w9WgXcQ" title="Terrain Generator Demo" */>}}

## The Problem

Hand-authored terrain takes days to iterate on. Level designers needed a way to rapidly prototype
biome layouts before committing to full sculpt time.

## Technical Implementation

```hlsl
float GenerateHeight(float2 uv, float scale, int octaves) {
    float value = 0.0;
    float amplitude = 0.5;
    for (int i = 0; i < octaves; i++) {
        value += snoise(uv * scale) * amplitude;
        scale  *= 2.0;
        amplitude *= 0.5;
    }
    return value;
}
```

### Architecture

- **Compute Shader** — generates heightmap data entirely on the GPU
- **C++ Actor** — manages LOD streaming and incremental mesh updates
- **Material Layer** — HLSL custom node blends biome textures based on temperature/humidity

### Key Features

1. **Multi-octave noise** — configurable octaves, frequency, and amplitude
2. **Biome blending** — temperature & humidity maps drive texture selection
3. **Runtime LOD** — quadtree subdivision reduces detail with distance
4. **Editor integration** — all parameters exposed in the Details panel

## Process

{{</* collapse summary="Full development log" */>}}
Started with a basic CPU Perlin implementation, profiled it, then moved heightmap generation to
a compute shader after CPU times exceeded 8ms per frame. Added biome blending in week 3 after
the initial prototype was approved by the art director.
{{</* /collapse */>}}

## Results

{{</* alert success */>}}
Generation time reduced from 240ms (CPU) to 6ms (GPU compute). Terrain detail increased 4× with no frame budget impact.
{{</* /alert */>}}

| Metric | Before | After |
|---|---|---|
| Generation time | 240ms | 6ms |
| Max triangle count | 50k | 200k |
| Memory footprint | 120 MB | 95 MB |

## Gallery

{{</* gallery images="/images/terrain-generator/overview.png, /images/terrain-generator/biomes.png, /images/terrain-generator/wireframe.png" */>}}

## Source Code

{{</* github-code user="MontyAnim" repo="ue5-terrain-gen" file="Source/TerrainGen/HeightmapCompute.usf" lang="hlsl" */>}}

{{</* button link="https://github.com/MontyAnim/ue5-terrain-gen" text="View Full Repository on GitHub" target="_blank" */>}}
```
