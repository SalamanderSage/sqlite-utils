# Repository Summary: sqlite-utils

**sqlite-utils** (v4.2.1, Apache-2.0, by Simon Willison) is a Python CLI tool and library for creating and manipulating SQLite databases: piping JSON/CSV/TSV into tables, in-memory SQL queries over files, full-text search, table transforms that `ALTER TABLE` can't do, column extraction, migrations, and a plugin system. Requires Python >= 3.10.

## Top-level layout

```
.
├── sqlite_utils/        # The package (~11k lines)
├── tests/               # pytest suite (~60 test modules)
├── docs/                # Sphinx docs (reStructuredText + a tutorial notebook)
├── .github/             # CI workflows and a custom action
├── pyproject.toml       # Packaging, dependencies, flake8 config
├── Justfile             # Dev task runner (test, lint, docs, cog)
├── mypy.ini, codecov.yml, .readthedocs.yaml, MANIFEST.in, LICENSE, README.md
└── (untracked) Dockerfile, docker-entrypoint.sh, requirements.txt,
    settings.json, statusline.sh   # local dev-container setup, not part of the project
```

## `sqlite_utils/` — the package

| File | Lines | Purpose |
|------|------:|---------|
| `db.py` | 5547 | Core Python API: `Database`, `Table`, `View` classes — create/insert/upsert/update/delete, transforms, extracts, FTS, foreign keys, m2m, indexes, etc. |
| `cli.py` | 3813 | Click-based `sqlite-utils` command (entry point `sqlite_utils.cli:cli`), wrapping the library API |
| `create_table_parser.py` | 897 | Parses `CREATE TABLE` SQL to introspect schemas |
| `utils.py` | 666 | Helpers: file/row reading (CSV/TSV/JSON sniffing), type suggestion, chunking, etc. |
| `migrations.py` | 184 | Python-file-based migration support (`sqlite-utils migrate`) |
| `recipes.py` | 76 | Built-in conversion functions for `convert` (e.g. date parsing, JSON splitting) |
| `plugins.py` / `hookspecs.py` | 35 / 17 | pluggy-based plugin manager and hook specifications |
| `__init__.py`, `__main__.py`, `py.typed` | — | Public exports, `python -m sqlite_utils`, typing marker |

## `tests/`

pytest suite largely organised one module per feature: CLI (`test_cli*.py`), inserts/upserts, transforms, extracts, FTS, GIS/SpatiaLite, migrations, plugins, introspection, WAL, atomicity, hypothesis-based property tests, and `test_docs.py` (checks docs stay in sync with code). Fixtures in `conftest.py`; `ext.c` is a test SQLite extension; `sniff/` holds sample CSVs for delimiter detection.

## `docs/`

Sphinx site (furo theme, published on Read the Docs): `cli.rst`, `cli-reference.rst`, `python-api.rst`, `reference.rst`, `migrations.rst`, `plugins.rst`, `installation.rst`, `upgrading.rst` (3.x → 4.0 guide), `contributing.rst`, `changelog.rst`, plus `tutorial.ipynb`. Parts of README and docs are generated with **cog**.

## CI (`.github/`)

Workflows: `test.yml` (matrix tests), `test-coverage.yml` (Codecov), `test-sqlite-support.yml` (multiple SQLite versions via the custom `actions/setup-sqlite-version` action), `publish.yml` (PyPI release), `codeql-analysis.yml`, `documentation-links.yml`, `spellcheck.yml`.

## Tooling & dependencies

- **Runtime deps:** click, click-default-group, pluggy, python-dateutil, sqlite-fts4, tabulate, pip.
- **Dev workflow:** `uv` + `just`. `just test` runs pytest; `just lint` runs black, flake8, mypy, pyright, ty, cog `--check`, and codespell; `just docs` serves live docs.

## Untracked local files

`Dockerfile`, `docker-entrypoint.sh`, `requirements.txt`, `settings.json`, and `statusline.sh` set up a Python 3.12 dev container with Claude Code, OpenCode, and ngrok, persist Claude credentials across runs, and configure a Claude Code status line. They aren't part of sqlite-utils, and their `requirements.txt` (streamlit, anthropic, fastapi, …) has nothing to do with the project's own dependencies.
