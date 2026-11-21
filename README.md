# PE Market Scout

Finds which design-only power electronics freelance projects are most in demand online,
and ranks opportunities by fit with the owner's expertise. See `CLAUDE.md` for rules and
`PLAN.md` for the phased plan.

## Status
Phase 0 (setup) and Phase 1 (problem definition) complete. No collectors yet.

## Dev quickstart
```bash
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest
ruff check .
```

## Docs
- `docs/taxonomy.md`: labels and keyword lists
- `docs/schema.md`: record schema (single source of truth)
- `docs/sources.md`: candidate sources, access method, ToS status
