# Diego Montúfar - Technical Artist Portfolio

A modern, responsive portfolio website built with Hugo and deployed on GitHub Pages.

## 🚀 Features

- **Custom Hero Section** with animated elements
- **Bimodal Project Grid** - Filter between Tools and Visual projects
- **Responsive Design** with dark mode support
- **Reusable Shortcodes** for embedding media:
  - Vimeo videos (clean, no chrome)
  - GitHub Gists
  - WebGL/Shader demos
- **Project Archetype System** for easy content creation
- **SEO Optimized** with proper meta tags
- **Fast Performance** with optimized assets

## 📁 Project Structure

```
MontyAnim.github.io/
├── archetypes/          # Content templates
│   ├── default.md
│   └── projects.md      # Project archetype with all fields
├── content/             # Site content
│   ├── about.md
│   ├── contact.md
│   ├── cv.md
│   └── projects/        # Project posts
├── layouts/             # Custom templates
│   ├── _default/        # Base layouts
│   ├── index.html       # Homepage
│   ├── projects/        # Project layouts
│   ├── about/           # About page layout
│   ├── cv/              # CV page layout
│   └── shortcodes/      # Reusable components
├── static/              # Static assets
│   ├── demos/           # WebGL/Shader demos
│   └── images/          # Images and thumbnails
├── themes/              # Hugo themes
│   └── PaperMod/
└── hugo.toml            # Site configuration
```

## 🛠️ Adding New Content

### Create a New Project

```bash
hugo new content/projects/my-project.md --kind projects
```

This creates a new project with the template including:
- Front matter (title, date, tech_stack, etc.)
- Placeholder sections
- Example shortcode usage

### Edit the Project

1. Update the front matter:
   - `title`: Project name
   - `tech_stack`: Array of technologies used
   - `project_type`: "tool" or "visual"
   - `thumbnail_url`: Path to thumbnail image
   - `hero_video_id`: Vimeo video ID (optional)
   - `summary`: Brief description

2. Add your content using markdown

3. Use shortcodes for rich media:
   ```markdown
   {{</* vimeo id="123456" */>}}
   {{</* gist MontyAnim GIST_ID */>}}
   {{</* webgl demo="shader-name" */>}}
   ```

4. Set `draft: false` when ready to publish

### Add Thumbnails

Place project thumbnails in `/static/images/thumbnails/`:
- Format: GIF (animated) or PNG/JPG (static)
- Size: 800x600px or 16:9 ratio
- Keep under 5MB

## 🎨 Shortcodes Reference

### Vimeo Video
```markdown
{{</* vimeo id="VIDEO_ID" title="Optional Title" */>}}
```

### GitHub Gist
```markdown
{{</* gist username GIST_ID */>}}
```

### WebGL/Shader Demo
```markdown
{{</* webgl demo="folder-name" title="Demo Title" height="600px" */>}}
```

## 🚀 Local Development

1. **Install Hugo** (Extended version):
   ```bash
   # Windows (using Chocolatey)
   choco install hugo-extended

   # macOS
   brew install hugo
   ```

2. **Clone the repository**:
   ```bash
   git clone https://github.com/MontyAnim/MontyAnim.github.io.git
   cd MontyAnim.github.io
   ```

3. **Initialize submodules**:
   ```bash
   git submodule update --init --recursive
   ```

4. **Run development server**:
   ```bash
   hugo server -D
   ```

5. **View at**: `http://localhost:1313`

## 📦 Deployment

The site auto-deploys to GitHub Pages via GitHub Actions when you push to the `main` branch.

### Manual Build

```bash
hugo --minify
```

Built files will be in the `/public` directory.

## 🎯 Workflow

1. **Work on feature branch** (e.g., `Publish`)
2. **Test locally** with `hugo server -D`
3. **Commit and push** to feature branch
4. **Create Pull Request** to `main`
5. **Merge PR** - GitHub Actions deploys automatically

## 🔧 Configuration

Edit `hugo.toml` to customize:
- Site title and description
- Social media links
- Menu items
- SEO settings

## 📝 Content Guidelines

### Projects
- Focus on problem-solution-results structure
- Include visual examples (screenshots, GIFs, videos)
- Add code snippets for technical projects
- Tag appropriately for filtering

### Writing Style
- Clear and concise
- Technical but accessible
- Show, don't just tell
- Include metrics when possible

## 🎨 Design System

### Colors
- Primary: Blue (#3b82f6)
- Secondary: Slate (#64748b)
- Automatic dark mode support

### Typography
- Headings: System font stack
- Code: Courier New, monospace

## 📄 License

Copyright © 2025 Diego Montúfar Pérez. All rights reserved.

## 🤝 Contact

- **Email**: diego.montufar@outlook.com
- **ArtStation**: [monty_anim](https://www.artstation.com/monty_anim)
- **GitHub**: [MontyAnim](https://github.com/MontyAnim)
