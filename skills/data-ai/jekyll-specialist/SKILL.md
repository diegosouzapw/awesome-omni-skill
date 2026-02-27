---
name: jekyll-specialist
description: Manages the samueltauil.github.io Jekyll site. Use for creating blog posts, updating photography content, editing the home page agent file, modifying styles, and maintaining site structure. Handles posts, photography galleries, resume updates, and GitHub Copilot dark theme customization.
metadata:
  author: samueltauil
  version: "1.0"
  site: samueltauil.github.io
compatibility: Requires Jekyll, GitHub Pages. Works with GitHub Copilot in VS Code.
---

# Jekyll Site Specialist

You are an expert assistant for managing the samueltauil.github.io Jekyll site—a personal portfolio with GitHub Copilot dark theme styling.

## Site Structure

```
_config.yml          # Site configuration (title, description, social links)
_layouts/            # Page templates (default.html, home.html, page.html, post.html)
_includes/           # Reusable components (header.html, footer.html)
_posts/              # Blog posts in YYYY-MM-DD-title.md format
assets/css/style.scss # Main stylesheet with GitHub Copilot theme
index.md             # Home page with agent-style code window
about.md             # Resume/About page
photography.md       # Analog photography portfolio
posts.md             # Blog listing page
resume.md            # Detailed professional resume
scripts/             # Automation scripts (update_lomography_photos.py)
.github/workflows/   # GitHub Actions (deployment, photo updates)
```

## Quick Reference

| Task | File |
|------|------|
| New blog post | `_posts/YYYY-MM-DD-title.md` |
| Update photos | `photography.md` |
| Edit home page | `index.md` |
| Update resume | `about.md` |
| Change styling | `assets/css/style.scss` |
| Edit header/nav | `_includes/header.html` |

## Creating Blog Posts

New posts go in `_posts/` with filename format: `YYYY-MM-DD-slug-title.md`

See [templates reference](references/TEMPLATES.md) for full post template.

### Common Categories
- `github-copilot` - GitHub Copilot content
- `vscode` - VS Code tips and extensions
- `devops` - CI/CD, automation, pipelines
- `github` - GitHub features and workflows
- `ai` - AI/ML and developer tools

### Common Tags
`github-copilot`, `vscode`, `github-actions`, `ci-cd`, `automation`, `devops`, `open-source`, `kubernetes`, `openshift`, `gitops`, `developer-tools`, `productivity`

## Photography Page

### Current Cameras
- MiNT SLR670-X Ming Edition
- Polaroid SX-70 Sonar
- Polaroid SLR 680se
- Lomo LC-A
- Canon AE-1 Program
- Pentax K1000
- Mamiya RB67

Photos are embedded from Lomography CDN. See [templates reference](references/TEMPLATES.md) for gallery and camera item HTML.

## Home Page Agent File

The home page (`index.md`) displays a styled code window that looks like a Copilot agent file. Uses inline styles for syntax highlighting:

- **Keys**: `color:#58a6ff` (blue)
- **Values**: `color:#ffa657` (orange)
- **Headers**: `color:#a371f7` (purple)
- **Descriptions**: `color:#7ee787` (green)
- **Comments**: `color:#6e7681` (gray)

See [styling reference](references/STYLING.md) for full color palette and CSS classes.

## Automated Workflows

### Weekly Lomography Photo Update
- Runs: Sundays at 6AM UTC
- Script: `scripts/update_lomography_photos.py`
- Fetches latest 12 photos from Lomography profile
- Auto-commits if new photos found

### Deployment
- Triggers on push to main branch
- Uses GitHub Pages with Jekyll

## Common Tasks

### Create a post from LinkedIn article
1. Copy article content
2. Create file: `_posts/YYYY-MM-DD-slug.md`
3. Add frontmatter with original post date
4. Format content with proper Markdown
5. Add LinkedIn attribution at bottom

### Update professional info
- Edit `about.md` for full resume
- Edit `resume.md` for detailed version
- Update `index.md` agent file for home page display

### Add a new certification
Add to `about.md` under appropriate category:
```markdown
- [Certification Name](https://www.credly.com) (Month Year)
```

### Update navigation
Edit `_includes/header.html` to add/reorder menu items.

### Change site metadata
Edit `_config.yml` for `title`, `description`, `github_username`, `linkedin_username`.
