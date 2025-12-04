# morce.codes

A personal blog and project showcase site built with Hugo and deployed via GitHub Pages.

## 🌐 Live Site

Visit the site at: [https://morce.codes](https://morce.codes)

## 🛠️ Tech Stack

### Core Technologies

- **[Hugo](https://gohugo.io/)** - Static Site Generator
  - Fast, flexible, and written in Go
  - Generates static HTML from Markdown content
  - Version: Latest (Extended edition)

- **[noJS Theme](https://git.andy.sb/nojs)** - Minimal Hugo Theme
  - No JavaScript required
  - Dark mode support via `prefers-color-scheme`
  - Clean, minimal design
  - Responsive layout

### Deployment & CI/CD

- **[GitHub Pages](https://pages.github.com/)** - Static Site Hosting
  - Custom domain: `morce.codes`
  - Automatic deployment via GitHub Actions

- **[GitHub Actions](https://github.com/features/actions)** - Continuous Integration/Deployment
  - Automated builds on push to `main` branch
  - Uses `peaceiris/actions-hugo@v3` for Hugo setup
  - Builds and deploys to GitHub Pages automatically

### Content & Structure

- **Markdown** - Content written in Markdown format
- **Goldmark** - Hugo's default Markdown renderer (with unsafe HTML enabled for custom styling)
- **Custom Shortcodes** - Image shortcode for flexible image rendering
- **Custom Layouts** - Override theme templates for homepage and post display

## 📁 Project Structure

```
morce.codes/
├── content/           # Markdown content files
│   └── posts/         # Blog posts
├── layouts/           # Custom Hugo templates
│   ├── _partials/     # Reusable template partials
│   ├── shortcodes/    # Custom shortcodes
│   └── home.html      # Homepage layout
├── static/            # Static assets (images, downloads, etc.)
│   ├── images/        # Image files
│   ├── downloads/     # Downloadable files
│   └── CNAME          # Custom domain configuration
├── themes/            # Hugo themes
│   └── nojs/          # noJS theme files
├── .github/
│   └── workflows/     # GitHub Actions workflows
│       └── gh-pages.yml
└── hugo.toml          # Hugo configuration
```

## 🚀 Getting Started

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) (required for the theme)

### Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/52454D434F/morce.codes-HUGO.git
   cd morce.codes-HUGO
   ```

2. Run Hugo development server:
   ```bash
   hugo server
   ```

3. Open your browser to `http://localhost:1313`

### Building for Production

```bash
hugo --minify
```

The generated site will be in the `public/` directory.

## 📝 Adding Content

### Creating a New Post

1. Create a new Markdown file in `content/posts/`
2. Add frontmatter with metadata:
   ```markdown
   +++
   date = '2025-12-01T14:23:59-06:00'
   draft = false
   title = 'Your Post Title'
   tags = ['tag1', 'tag2']
   image = 'images/your-image.png'
   +++
   
   Your post content here...
   ```

### Adding Images

Place images in `static/images/` and reference them using the custom shortcode:

```markdown
{{< img src="/images/your-image.png" alt="Description" style="width: 256px;" >}}
```

Or use standard Markdown:
```markdown
![Alt text](/images/your-image.png)
```

## 🔧 Configuration

Key configuration is in `hugo.toml`:

- **baseURL**: Set to `https://morce.codes/` for production
- **theme**: Uses `nojs` theme
- **markup**: Goldmark renderer with unsafe HTML enabled for custom styling
- **menus**: Navigation menu configuration

## 🌍 Custom Domain

The site uses a custom domain `morce.codes` configured via:
- `static/CNAME` file containing the domain name
- GitHub Pages custom domain settings

## 📦 Features

- ✅ Automatic deployment on push to `main` branch
- ✅ Custom domain support
- ✅ Tag-based post organization
- ✅ Full post display on homepage
- ✅ Responsive design
- ✅ Dark mode support
- ✅ No JavaScript required
- ✅ Fast static site generation

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 🙏 Credits

- **Hugo** - Static site generator
- **noJS Theme** - Minimal Hugo theme by [Andy Sukowski-Bang](https://andy.sb)
- **GitHub Pages** - Hosting platform
- **GitHub Actions** - CI/CD automation

---

Built with ❤️ using Hugo and deployed via GitHub Pages.

