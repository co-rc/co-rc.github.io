# Generating pages

## Github actions

- Push to `main`
- GitHub Actions builds and deploys to GitHub Pages
- Result: https://co-rc.github.io/

## Local preview (optional)

### Windows

```bash
python -m venv .venv
.venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

### Linux

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```
