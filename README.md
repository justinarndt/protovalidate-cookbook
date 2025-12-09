# Protovalidate Patterns Cookbook

A practical guide to implementing validation patterns with [protovalidate](https://github.com/bufbuild/protovalidate), Buf's runtime validation library for Protocol Buffers.

## 🚀 Live Demo

View the documentation at: https://justinarndt.github.io/protovalidate-cookbook

## 📖 Contents

- **Getting Started**: Overview and quickstart guide
- **Patterns**: 
  - String validation (email, UUID, regex)
  - Numeric constraints (ranges, bounds)
  - Enum validation (defined_only, value restrictions)
  - Message-level rules (cross-field validation)
  - Custom CEL expressions
- **Reference**:
  - Error handling best practices
  - Troubleshooting guide

## 🛠 Local Development

```bash
# Install dependencies
pip install mkdocs mkdocs-material

# Serve locally with hot reload
mkdocs serve

# Build static site
mkdocs build
```

## 🚢 Deployment

This site deploys automatically to GitHub Pages when you push to `main`. 

Manual deployment:
```bash
mkdocs gh-deploy
```

## 📝 About

This cookbook was created as a technical writing sample demonstrating:
- MkDocs with Material theme
- Developer-focused documentation
- Practical validation patterns for protovalidate

## 📚 Resources

- [Protovalidate Documentation](https://buf.build/docs/protovalidate/)
- [Protovalidate GitHub](https://github.com/bufbuild/protovalidate)
- [Buf CLI](https://buf.build/docs/installation)
- [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)

## 📄 License

MIT License - feel free to use as a reference for your own documentation projects.

---

*Built by [Justin Arndt](https://github.com/justinarndt)*
