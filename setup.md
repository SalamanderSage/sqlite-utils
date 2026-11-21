# How to run claude agent safely.

# Build the image using the Dockerfile, requirements.txt, settings.json, statusline.sh, and docker-entrypoint.sh
The image contains Python 3.12, uv (package manager), just (command runner), sqlite3, git, Node.js

docker build -t sqlite-utils-image .



# Installing dependencies
running it this way allows the container to access the internet which we dont want while we our working but we need to run it once this way to install the dependencies which will persist throughout containers so that future runs will not need network access

docker run -it --rm -v "${PWD}:/workspace" -v sqlite-utils-venv:/opt/venv -w /workspace sqlite-utils-image

then download the dependencies by running 
uv sync
exit

# Running the container
for all future runs the container should be run using this prevents the container from accessing the internet and other parts of the filesystem except the mounted folder and the dependecies in /opt/venv

docker run -it --rm --network none -v "${PWD}:/workspace" -v sqlite-utils-venv:/opt/venv -e UV_OFFLINE=1 -w /workspace sqlite-utils-image



# Agent Smoke Test

Run the containter with 
docker run -it --rm --network none -v "${PWD}:/workspace" -v sqlite-utils-venv:/opt/venv -e UV_OFFLINE=1 -w /workspace sqlite-utils-image

Confirm the container has no network access with
curl -I https://www.google.com
it should return:
    curl: (6) Could not resolve host: www.google.com

Make sure the container cant access your unmounted filesystem which could contain sensitive information or credentials by using the following commands
ls /home
it should return nothing
grep -vE 'proc|sysfs|tmpfs|cgroup|devpts|mqueue' /proc/mounts
it should only return: 
    overlay / overlay ...
    /dev/sdf /workspace ext4 rw,relatime,discard,errors=remount-ro,data=ordered 0 0
    /dev/sdf /opt/venv ext4 rw,nosuid,nodev,relatime,discard,errors=remount-ro,data=ordered 0 0

Then run claude with 
claude

ask claude to summarize the repo structure and write the summary to /workspace/agent-summary.md
make sure claude wrote it with cat agent-summary.md

Then exit the container and make sure the file persists outside the container with
exit
cat agent-summary.md

The output of this will be something like this:

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


# Security decisions

- only the project folder is mounted preventing the container and agent from accessing the host computers file system
- no credentials are in the container for the agent to see
- The container and all temporary files created while working in it are kept ephemeral by using --rm in the docker run command.
- the container and agent do not have netowrk access
- he repository folder persists allowing any files the agent creates or updates to keep their changes.
- packages are kept outside the container so that they do not need to be reinstalled with each run since they cant be without network access

What dependencies did you include in your extended Docker image?
- I extended the docker image by adding uv for package management, just for testing, sqlite3 for database management. On the first run of the container with network uv sync is used to download some other packages such as click, pytest, black, and more

What did your smoket test prove?
- Asking the agent to write a summary on the project and write it the workspace proves that the agent can see all the information it needs to and making sure the output persists on the host machine proves the agents work persists outside the container. Running the container and making sure ls /opt/venv and ls /home confirms that the container cannot access files outside of the project workspace.

What risks remain?
- There could be a prompt injection attack by hiding commands for ai agents in the repository. The ai agent can still make mistakes and make bad changes or delete important files in the repository.