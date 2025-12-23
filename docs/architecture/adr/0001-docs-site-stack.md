# ADR 0001: Docs site stack (MkDocs + Material)

- Status: accepted
- Date: 2025-12-23

## Context

We need a public documentation site for the organization with:

- Markdown authoring
- Mermaid diagrams
- Fast onboarding and minimal toolchain

## Decision

Use:

- MkDocs
- Material for MkDocs
- GitHub Actions deployment to GitHub Pages

## Consequences

- Documentation lives centrally in `co-rc.github.io`.
- Diagrams are authored as Mermaid fenced blocks.
- Builds run in CI; local preview is optional (`mkdocs serve`).
