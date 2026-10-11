# nakomis-workflows

Reusable GitHub Actions workflows (`workflow_call`) shared by nakomis repos. Callers pin `@main`, so a
change here affects every caller's next run: keep inputs backwards compatible, add new inputs with defaults.

## Repository layout

- `.github/workflows/` — the reusable workflows (see README)
- `docs/architecture/` — diagrams, if any

## Testing

No unit tests. Validate with `actionlint` if available, and by running a caller (nakomis-shell or
nakomis-fleet CI) after changes.
