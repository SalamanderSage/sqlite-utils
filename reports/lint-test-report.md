# Lint & Test Report

Date: 2026-11-24 · Environment: Python 3.12.16, `uv`, ty 0.0.109

## Summary

| Check | Command | Result |
|---|---|---|
| Tests | `uv run pytest` | ✅ 1497 passed, 16 skipped, 33 warnings (9.7s) |
| Formatting | `uv run black . --check` | ✅ 65 files unchanged |
| Style | `uv run flake8` | ✅ no issues |
| Types (mypy) | `uv run mypy sqlite_utils tests` | ✅ no issues in 64 files |
| Types (pyright) | `uv run pyright sqlite_utils tests` | ✅ 0 errors, 0 warnings |
| Types (ty) | `uv run ty check sqlite_utils` | ❌ 1 diagnostic (exit 1) |
| Docs sync | `uv run cog --check README.md docs/*.rst` | ✅ all 12 files up to date |
| Spelling (docs) | `codespell docs/*.rst` | ✅ no issues |
| Spelling (source) | `codespell sqlite_utils` | ✅ no issues |

Overall: all tests pass. The only failing check is `ty`, with one warning-level diagnostic. Because `ty` exits non-zero, `just lint` would stop at that step.

## Test suite

- **1497 passed, 0 failed, 16 skipped.**
- **Skips** (all because optional dependencies are missing, not because of code problems):
  - 12 in `tests/test_gis.py`: SpatiaLite extension not found
  - 3 in `tests/test_cli.py:2850`: needs the compiled `ext.c` extension
  - 1 in `tests/test_create.py:1184`: pandas/numpy not installed
- **Warnings:** all 33 are the same `DeprecationWarning` from Click: `CliRunner.isolated_filesystem()` will be removed in Click 9.0. They come from 22 call sites, 19 in `tests/test_cli.py` and 3 in `tests/test_insert_files.py`. Nothing breaks today, but these tests will fail once Click 9 is released. Click suggests switching to pytest's `tmp_path` fixture.

## Linter findings

### ty: redundant cast (the one failure)

```
warning[redundant-cast]: Value is already of type `list[str]`
    --> sqlite_utils/db.py:4792:28
4792 |             column_names = cast(list[str], list(first_record))
help: Remove the redundant `cast`
```

The fix is a one-line change: `column_names = list(first_record)`. This report does not apply it, because source files were out of scope for this task.

### Minor notes (non-failing)

- mypy: `mypy.ini` has unused sections `[mypy-pysqlite3.*]` and `[mypy-sqlite_dump.*]`. This is informational only.
- black: `.ipynb` files were skipped because `black[jupyter]` is not installed.

## Notes on the run

- No files were modified. black ran with `--check` only.
- `git status` / `git diff` could not be run. The worktree's `.git` file points to `/home/salamander/LaunchCodeAgenticEngineer/sqlite-utils/.git/worktrees/target-agent-b`, which does not exist inside this environment.
