# Runbook

Operational recipes and troubleshooting.

## Local preview (optional)

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

## Publish flow

- Push to `main`
- GitHub Actions builds and deploys to GitHub Pages

## Troubleshooting

### Pages does not update

- Check Actions run status for the latest push.
- Confirm repository **Settings → Pages** uses **Source: GitHub Actions**.
- Confirm default branch is `main` (or update workflow to match).

### Mermaid diagrams not rendered

- Ensure fenced blocks use:

```text
```mermaid
...
```

- Ensure `mkdocs.yml` includes `pymdownx.superfences` with the `mermaid` custom fence.
