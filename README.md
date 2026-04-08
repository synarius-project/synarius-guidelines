# Synarius programming guidelines

Cross-repository conventions for code and documentation in the **Synarius** project (Python, Qt/PySide6 where applicable).

## Where to read

- **HTML documentation (recommended):** after [GitHub Pages](https://pages.github.com/) is enabled for this repository, the built Sphinx site is published at  
  **https://synarius-project.github.io/synarius-guidelines/**  
  Start at the **home page**, then open **Programming guidelines** in the navigation.
- **Source repository:** [synarius-project/synarius-guidelines](https://github.com/synarius-project/synarius-guidelines)

Product-specific context (install, features, architecture) remains in each repository:

| Repository | Role |
|------------|------|
| [synarius-core](https://github.com/synarius-project/synarius-core) | GUI-less simulation backend |
| [synarius-apps](https://github.com/synarius-project/synarius-apps) | DataViewer, ParaWiz, shared Qt tools (usable **without** Synarius Studio) |
| [synarius-studio](https://github.com/synarius-project/synarius-studio) | Graphical modeling and simulation IDE |

## Build docs locally

```bash
pip install sphinx zerovm-sphinx-theme
sphinx-build -b html docs docs/_build/html
```

Open `docs/_build/html/index.html` in a browser.

## Publishing on GitHub

1. Create **synarius-project/synarius-guidelines** and push this tree.
2. In **Settings → Pages**, set the source to **GitHub Actions** (the workflow in `.github/workflows/docs.yml` deploys the Sphinx HTML).
3. After the first successful run, the public URL is typically `https://synarius-project.github.io/synarius-guidelines/`. Until then, links from other repos may return 404.

## License

See [LICENSE](LICENSE) (Mozilla Public License 2.0).
