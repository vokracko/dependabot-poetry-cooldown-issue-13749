# Repro: Dependabot poetry cooldown bypassed by caret resolution

Minimal reproduction for
[dependabot/dependabot-core#13749](https://github.com/dependabot/dependabot-core/issues/13749).

**Symptom:** Dependabot opens a PR titled *"Bump pytest-reportportal from 5.5.2 to 5.6.7"*
but the resulting `poetry.lock` contains **5.6.8** — a release inside the 7-day `cooldown`
window that should have been excluded.

## Key detail — the dependency is in a poetry GROUP

`pytest-reportportal` is declared in **`[tool.poetry.group.dev.dependencies]`**, not the
top-level `[tool.poetry.dependencies]` table. The cooldown bypass reproduces in this layout.
If the same dependency is moved to `[tool.poetry.dependencies]`, the resulting `poetry.lock`
correctly stays at 5.6.7 and the bug does **not** appear — so the group declaration is essential
to this reproduction.

## Files in this repo

| File | Role |
|------|------|
| `pyproject.toml` | `pytest-reportportal = "^5.5.2"` in **`[tool.poetry.group.dev.dependencies]`** (pre-update state) |
| `poetry.lock` | Locked at `5.5.2` (content-hash matches the `^5.5.2` pyproject) |
| `.github/dependabot.yml` | `pip` ecosystem with `cooldown.default-days: 7` |

## Release timeline

`pytest-reportportal` upload dates on PyPI:

| version | uploaded   | vs a 7-day cooldown evaluated ~2026-06-05 |
|---------|------------|-------------------------------------------|
| 5.5.2   | 2025-07-08 | starting pin                              |
| 5.6.7   | 2026-04-15 | ~51 days old → **allowed**                |
| 5.6.8   | 2026-06-03 | ~2 days old → **must be excluded**        |

The bug only manifests while 5.6.8 is still inside the cooldown window (i.e. within 7 days of
2026-06-03). After that, poetry resolving to 5.6.8 is no longer a cooldown violation.
