# WebGL/Shader Demos

Place your interactive shader demos in subfolders here. Each demo should have its own folder with an `index.html` file.

## Structure

```
demos/
├── my-shader-demo/
│   ├── index.html
│   ├── shader.js
│   └── assets/
└── another-demo/
    └── index.html
```

## Usage

Use the `webgl` shortcode in your markdown:

```
{{</* webgl demo="my-shader-demo" title="Cool Shader Effect" height="800px" */>}}
```
