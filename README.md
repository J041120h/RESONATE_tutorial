# RESONATE documentation site

MkDocs source of the RESONATE tutorial and documentation website,
published at **https://j041120h.github.io/RESONATE_tutorial/**.

The software lives in its own repository: https://github.com/J041120h/RESONATE.

## Layout

```
mkdocs.yml                  site configuration (theme, navigation, Markdown extensions, notebook plugin)
requirements-docs.txt       exact versions used to build the site
overrides/main.html         adds a download button to notebook pages
docs/                       page sources
  index.md                  home page
  installation.md
  tutorials/
    walkthrough.ipynb       the end-to-end tutorial: an executed notebook, shown as is (never re-executed by the build)
    cli.md                  the same workflow from a shell
    inputs_outputs.md, choosing.md
  api/                      hand-written API reference
  faq.md, citation.md
  resource/                 figures shown on the pages
  assets/                   stylesheet (extra.css) and the MathJax configuration
.github/workflows/deploy-docs.yml   builds with `mkdocs build --strict` and deploys to GitHub Pages
index.html, .nojekyll       only for the branch-deployment fallback below
```

The walkthrough notebook was run top to bottom in a new environment (RESONATE installed from GitHub, CPU PyTorch,
4 CPU cores); the outputs and figures in it come from that run (the figures were re-encoded as 128-colour PNGs to keep
the page light; nothing else was changed). The commands on the command-line page and the code on
the embeddings page were run the same way, and their printed outputs are pasted from those runs. To update the
walkthrough, run the notebook again in an empty folder and replace `docs/tutorials/walkthrough.ipynb` with the executed
copy.

## Local preview

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-docs.txt
mkdocs serve
```

Then open `http://127.0.0.1:8000`. `mkdocs build --strict` writes the static site to `site/` and fails on any broken
link or missing page.

## Deploy on GitHub Pages

### Preferred: GitHub Actions

1. Open the repository on GitHub.
2. Go to `Settings` -> `Pages`.
3. Under **Build and deployment**, set **Source** to `GitHub Actions`.
4. Push to `main` (or run the `deploy-docs` workflow by hand from the Actions tab).

The workflow builds the site and publishes the generated HTML at the GitHub Pages address.

If the deploy job fails with `HttpError: Not Found`: confirm that **Source** is `GitHub Actions`, that the repository's
Actions permissions allow `GITHUB_TOKEN` to write Pages deployments, and re-run the workflow (the first run after
enabling Pages can fail once).

### Fallback: branch deployment

1. Run `mkdocs build`.
2. Remove `site/` from `.gitignore` and commit the generated `site/` folder.
3. Keep the root `index.html`, which redirects to `./site/`.
4. In `Settings` -> `Pages`, deploy from the branch root.
