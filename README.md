# AGG AI Proxy API Documentation

This repository contains the official documentation for the AGG AI Proxy API, built with MkDocs Material.

## Building Documentation Locally

1. Install dependencies:

```bash
uv pip install -r pyproject.toml
```

2. Serve documentation locally:

```bash
mkdocs serve
```

3. Visit http://127.0.0.1:8000

## Building for Production

```bash
mkdocs build
```

The static site will be generated in the `site/` directory.

## GitHub Pages Deployment

Documentation is automatically deployed to GitHub Pages when changes are pushed to the `main` branch.

The deployment workflow:
- Triggers on changes to `docs/**`, `mkdocs.yml`, or the workflow file
- Builds the MkDocs site
- Deploys to GitHub Pages

## Documentation Structure

```
docs/
├── index.md              # Home page
├── quickstart.md         # Getting started guide
├── authentication.md     # Authentication guide
├── api/
│   ├── conversation.md   # Conversation API reference
│   ├── tools.md          # Tools API reference
│   └── files.md          # Files API reference
└── examples/
    ├── basic.md          # Basic usage examples
    └── tools.md          # Tool execution examples
```

## Configuration

Documentation configuration is in `mkdocs.yml`. It uses the Material theme with:
- Dark/light mode toggle
- Navigation tabs and sections
- Search functionality
- Code highlighting
- Syntax highlighting for multiple languages

## Contributing

When adding new documentation:
1. Create markdown files in the `docs/` directory
2. Update navigation in `mkdocs.yml` if needed
3. Test locally with `mkdocs serve`
4. Commit and push to trigger deployment

