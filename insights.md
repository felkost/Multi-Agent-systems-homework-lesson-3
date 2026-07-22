# insights.md — MA_systems_hl3 (repo root)

> Append-only. Add new entries at the bottom of the correct section.
> Discovery bar: "Would a fresh agent save ≥10 minutes from reading this?" If not, skip.
> Format: `**YYYY-MM-DD [Category]** — actionable sentence. \`file:line\``
> See the `sdd-engineering:engineering-insights` skill for full criteria and format rules.

## Patterns
<!-- Reusable approaches that worked in this module. -->

## Mistakes
<!-- Failure modes, antipatterns, wrong assumptions. Prioritize this section. -->

## Decisions
<!-- Architectural or design choices with the reasoning behind them. -->

- **2026-07-22 [Decision]** — `requirements.txt`'s five original dependency lines (langchain, ddgs, trafilatura, pydantic, pydantic-settings) intentionally keep `>=` per an explicit user decision, even though the assignment brief requires exact versions — do not "fix" them to `==`. Every dependency added afterward (e.g. `langchain-openai==1.4.0`, and everything in `requirements-dev.txt`) must be pinned exactly, resolved from a real `pip install` in `.venv`, never guessed. `requirements.txt`

## Quirks
<!-- Dependency gotchas, env constraints, non-obvious tool or library behavior. -->

- **2026-07-22 [Quirk]** — Git cannot re-include a file with `!pattern` once a parent directory is already excluded; a bare `output/` rule silently blocks `!output/.gitkeep` even though the negation line looks correct. Ignore the contents instead of the directory: `output/*` + `!output/.gitkeep`. `.gitignore`
- **2026-07-22 [Quirk]** — mypy skips type-checking the body of any function with zero type annotations by default, so bugs inside it stay invisible until the function is annotated. Adding `-> None` to a previously bare `def main():` made mypy check its body for the first time and surfaced a pre-existing error (`agent.stream()` called on `Ellipsis`, since `agent.py` is still `agent = ...`). `main.py:4`
- **2026-07-22 [Quirk]** — flake8 does not read `pyproject.toml` (requires a separate `.flake8`), and its `exclude` option replaces the default ignore list rather than extending it — omitting `.git`/`.venv`/cache dirs from a custom `exclude` re-exposes them to linting. `.flake8`
- **2026-07-22 [Quirk]** — pytest exits with code 5 (not a generic non-zero) when zero tests are collected, which fails CI on every push before any test file exists. Guard the CI step with `pytest || [ "$?" -eq 5 ]` to tolerate the empty-suite state without masking real test failures. `.github/workflows/ci.yml`

## Open Questions
<!-- Unresolved. Convert to an entry in the appropriate section when answered. -->

---
Last updated: 2026-07-22 · Entries: 5
