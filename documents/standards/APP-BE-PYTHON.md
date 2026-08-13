# Python Module Guide

> **Audience**: LLMs and developers writing Python modules and jobs in this codebase.
> **Stack**: Python 3.12+ · `uv` · `py-logging` · `py-db-migrate` · `psycopg2`
> **Architecture contract**: `APP-BE-PATTERNS.md` — read that first for the language-agnostic module structure, domain quartet, CRUD rules, and validation ordering this guide implements in Python.

---

## Dependency management — always use uv

Every Python module has a `pyproject.toml`. Never use `requirements.txt` for new modules.

```toml
[project]
name = "my-module"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32",
    "py-logging",
    "py-db-migrate",
]

[tool.uv]
package = false

[tool.uv.sources]
py-logging    = { git = "git+ssh://git@github.com/rohitphular/meridian-common-libs.git", subdirectory = "py-logging" }
py-db-migrate = { git = "git+ssh://git@github.com/rohitphular/meridian-common-libs.git", subdirectory = "py-db-migrate" }
```

Install / sync:
```bash
cd <module>/
uv sync          # creates .venv and installs all deps
uv run python runner.py
```

---

## Shared libraries — meridian-common-libs

Two libraries are mandatory in all new Python modules. Both are consumed via uv git sources from `git+ssh://git@github.com/rohitphular/meridian-common-libs.git`.

### `py-logging`

Structured logger. Always use it — never use `print()`, `logging.basicConfig()`, or bare `logging.getLogger()`.

```python
from py_logging import get_logger
logger = get_logger(__name__)
```

Requires `MERIDIAN_LOG_ROOT` env var set to the absolute path of the logs directory. Raises `EnvironmentError` at import time if not set — do not suppress this.

Log format: `[YYYY-MM-DD HH:MM:SS UTC] [LEVEL] [module.path] message`

Log files: `$MERIDIAN_LOG_ROOT/{top_module}/{top_module}.log` — daily rotation, previous day deleted.

### `py-db-migrate`

Schema migration runner. Supports PostgreSQL and ClickHouse. Used for all DDL — never run `CREATE TABLE` ad-hoc.

Database drivers are optional extras — declare the one you need:

```toml
[project]
dependencies = [
    "py-db-migrate[postgres]",    # for PostgreSQL
    # or
    "py-db-migrate[clickhouse]",  # for ClickHouse
]

[tool.uv.sources]
py-db-migrate = { git = "git+ssh://git@github.com/rohitphular/meridian-common-libs.git", subdirectory = "py-db-migrate" }
```

Never declare bare `py-db-migrate` without an extra — the adapter will raise `ImportError` at runtime with a message naming the missing extra.

```python
from py_db_migrate.core.config import ConnectionConfig
from py_db_migrate.adapters.postgres import get_client, ensure_schema_migration_table

config = ConnectionConfig(host=..., port=..., user=..., password=..., connect_database=...)
client = get_client(config)
```

Migration files live in `migrations/`. Naming: `NNNN_description.py` — four-digit zero-padded sequence, `snake_case` description.

```python
# migrations/0001_create_records.py
def upgrade(client) -> None:
    with client.cursor() as cursor:
        cursor.execute("CREATE TABLE IF NOT EXISTS ...")
    client.commit()
```

---

## Module types

Two distinct Python module types exist in this codebase. Apply the correct pattern for each — the folder structure, `__init__.py` rules, and `pyproject.toml` shape differ between them.

| Type | Purpose | Consumed by |
|---|---|---|
| **Job module** | Runs as a process — scheduled, triggered, or invoked manually | Scheduler, cron, `make` targets |
| **Library package** | Installed as a dependency; provides a reusable importable API | Other modules via `pyproject.toml` uv git source |

---

## Job module — folder structure

```
<module>/
  _tasks/              ← design and task docs for this module
  pyproject.toml       ← uv deps (always — no requirements.txt)
  py_db_migrate.toml   ← migration CLI config
  runner.py            ← entry point
  config.py            ← DB config + external API keys from env vars
  fetcher.py           ← main job class
  sources/             ← one file per external data source, named after the source
    <source_a>.py
    <source_b>.py
  database/
    upsert.py          ← PostgreSQL upsert helpers
  migrations/
    0001_<name>.py
    0002_<name>.py
```

No `__init__.py` files anywhere in a job module. Python 3.3+ namespace packages handle directory imports without them. Never add `__init__.py` as a way to make a directory importable — import explicitly by module path instead.

Job modules set `package = false` in `pyproject.toml` — they are not installed as packages:

```toml
[tool.uv]
package = false
```

---

## Library package — folder structure

Use the `src` layout. Placing the package under `src/` keeps the uninstalled source tree off the Python path during development and test runs — without it, `import <package>` resolves to the local directory rather than the installed package, silently masking packaging errors.

```
<lib-name>/
  src/
    <package_name>/
      __init__.py        ← public API — re-exports only what consumers should import
      <module_a>.py      ← implementation file; name describes what it does, not what it is
      <module_b>.py
      py.typed           ← PEP 561 marker (required for typed packages)
  tests/
    unit/
    integration/
  Makefile             ← `make test` (+ `make test-unit` / `make test-integration` when split)
  pyproject.toml
```

**`__init__.py` in library packages:** Library packages use `__init__.py` to define a stable public API contract. It must only re-export the symbols consumers are expected to import — no internal implementation details. This is the only context where `__init__.py` is used in this codebase.

**File naming inside a library package:** Every implementation file must be named after what it does, not after a generic role. `sheets_client.py` is correct; `client.py` is not — it says nothing about which client.

**`pyproject.toml` for a library package:**

```toml
[project]
name = "my-lib"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "some-dep>=1.0",
    "py-logging",
]

[tool.uv.sources]
py-logging = { git = "git+ssh://git@github.com/rohitphular/meridian-common-libs.git", subdirectory = "py-logging" }

[dependency-groups]
dev = [
    "pytest>=8.0",
    "ruff>=0.9",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/<package_name>"]
```

Library packages do **not** set `[tool.uv] package = false` — that flag is for job modules only.

Library packages do **not** commit a `uv.lock`. A lockfile pins exact dependency versions for a reproducible install, which is a job-module concern — a library's consumers resolve and lock their own dependency versions. Add `uv.lock` to `.gitignore` in the library repo. Job modules, by contrast, do commit their `uv.lock`.

**Consuming a library package** in another module's `pyproject.toml`:

```toml
[tool.uv.sources]
my-lib = { git = "git+ssh://git@github.com/rohitphular/meridian-common-libs.git", subdirectory = "my-lib" }
```

**Importing from a library package:**

```python
from my_lib import PublicClass   # resolved via __init__.py re-export
```

---

## Linting and formatting — always use ruff

Every Python module — job module and library package alike — uses `ruff` for both linting and formatting. There is no separate linter or formatter (no `flake8`, `black`, `isort`, or `pylint`); `ruff` covers all of them.

`ruff` is declared under `[dependency-groups] dev` (shown in the pyproject examples above) and configured with this exact block in every `pyproject.toml`:

```toml
[tool.ruff]
line-length = 200
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I"]

[tool.ruff.format]
quote-style = "double"
```

- `line-length = 200` — the project line-length limit.
- `target-version = "py312"` — matches `requires-python = ">=3.12"`.
- `select = ["E", "F", "I"]` — pycodestyle errors (`E`), pyflakes (`F`), and import sorting (`I`). Import order is enforced by the linter — never hand-sort or hand-group imports. Shared internal libraries are treated as third-party by default; do not insert a blank line to separate them into their own group. To give internal packages a dedicated import group, set them in `[tool.ruff.lint.isort] known-first-party` — never simulate it with manual spacing.
- `quote-style = "double"` — double quotes everywhere.

Run both commands from the module root before every commit. Both must pass with zero findings — a module with lint or format violations is not shippable:

```bash
uv run ruff check .          # lint — no violations allowed
uv run ruff format --check .  # formatting — must already be formatted
```

`ruff check .` reports lint violations; `ruff format --check .` fails if any file is not already formatted. To fix rather than check, run `uv run ruff check --fix .` and `uv run ruff format .`.

`ruff format` also normalises Python code blocks embedded in Markdown (`.md`) files, so documentation examples are held to the same formatting as source.

These two commands are the mandatory lint/format gate in CI — see `APP-CICD-BE-PYTHON.md`.

---

## Testing

The language-agnostic testing rules are in `APP-BE-PATTERNS.md § Testing`. Python specifics:

- **Layout** — unit tests in `tests/unit/`, integration tests in `tests/integration/`. No test file sits directly in `tests/`, and there are no `__init__.py` files under `tests/` (pytest discovers by path).
- **Runner** — `pytest`, declared under `[dependency-groups] dev`. Run `uv run pytest tests/unit/` for the fast suite and `uv run pytest tests/integration/` for the integration suite.
- **Makefile** — every library ships a `Makefile` with a `make test` target that runs its suite via `uv run pytest`. When the library has a unit/integration split, also provide `make test-unit` (fast, no external services) and `make test-integration`; `make test` runs the full suite. The full `make test` may require Docker (containerised integration tests) or credentials — `make test-unit` is the dependency-free path, and always passes without external services. The repository root ships a generic `Makefile` whose `make test` runs `make test` in every subdirectory that has one, so new libraries are covered automatically.
- **Unit tests** — no network, no database, no containers. Use the `tmp_path` and `monkeypatch` fixtures for file-system and environment needs; never touch a real external service.
- **Integration tests** — exercise the real dependency in a disposable container (e.g. `testcontainers`); never mock the client under test. Start the container once per module with a `scope="module"` fixture, not once per test.
- **Skips** — an integration test whose dependency is unavailable (an optional-dependency extra not installed, or no container runtime) calls `pytest.skip(...)` with a clear reason instead of failing.
- **Shared fixtures** — put shared setup in `conftest.py`; do not import it from other test files.
- **Naming** — test function names describe the behaviour verified (`test_read_missing_sheet_returns_empty`), not the mechanism.

---

## Config from env vars

All configuration reads from environment variables — no hardcoded values.

```python
# config.py
import os
from py_db_migrate.core.config import ConnectionConfig

def db_config() -> ConnectionConfig:
    return ConnectionConfig(
        host=os.environ["DB_HOST"],
        port=int(os.environ.get("DB_PORT", "5432")),
        user=os.environ["DB_USER"],
        password=os.environ["DB_PASSWORD"],
        connect_database=os.environ["DB_NAME"],
    )
```

Raise `EnvironmentError` immediately if a required variable is absent — never silently fall back.

---

## Running migrations

`py_db_migrate.toml` at the module root holds the connection config (reads from env vars). Apply all pending migrations with:

```bash
cd <module>/
export $(grep -v '^#' .env | xargs)
uv run py-db-migrate run --db postgres
```

---

## Runner pattern

```python
# runner.py
import argparse
from py_logging import get_logger
import config
from fetcher import MyJob

logger = get_logger(__name__)

def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument('--backfill', metavar='YYYY-MM-DD')
    args = parser.parse_args()
    job = MyJob(config.db_config())
    if args.backfill:
        logger.info(f"runner: mode=backfill from_date={args.backfill}")
        job.backfill(args.backfill)
    else:
        logger.info("runner: mode=daily")
        job.run()

if __name__ == '__main__':
    main()
```

---

## Job class pattern

```python
# fetcher.py
import sources.source_a as source_a
import sources.source_b as source_b
from database.upsert import upsert_records
from py_db_migrate.core.config import ConnectionConfig
from py_db_migrate.adapters.postgres import get_client
from py_logging import get_logger

logger = get_logger(__name__)


class MyJob:
    def __init__(self, db: ConnectionConfig) -> None:
        self._db = db

    def run(self) -> None:
        client = get_client(self._db)
        try:
            data = self._fetch()
            self._upsert(client, data)
            logger.info(f"job: rows={len(data)}")
        finally:
            client.close()

    def backfill(self, from_date: str) -> None:
        client = get_client(self._db)
        try:
            ...
        finally:
            client.close()

    def _fetch(self): ...
    def _upsert(self, client, data): ...
```

Always close the client in a `finally` block. Use `upsert` patterns with `ON CONFLICT` — never truncate-and-reload PostgreSQL tables.

---

## Coding rules

### Data handling

- External data source values may be untyped — always cast explicitly: `float(row['amount'])`, `int(row['count'])`.
- Missing or empty values: always `row.get('field') or default`.
- Date strings: `datetime.fromisoformat(row['date_field'])`.
- PostgreSQL `NUMERIC` values come back as `Decimal` — convert to `float` only at display time, not in computation.

### Logging

Use `py-logging` exclusively. Format: `fnname: key=value key=value`.

```python
logger.info(f"run: input_rows={len(rows)} date={today}")
logger.warning(f"run: skipped_rows={skipped} reason=missing_value")
logger.error(f"run: error={e}")
```

Log at the start of `run()` with input counts, at the end with output counts. Never log:
- API keys, DB passwords, or any secret loaded from env
- Raw sensitive row data
- Any value from `.env` that is a secret

### Keep `run()` thin

`run()` orchestrates: read → compute → write. All computation lives in private `_methods` — pure functions with no I/O. This makes jobs testable without a real external connection.

### Error handling

Jobs catch all exceptions, log them, and exit with code 1. The runner should not suppress exceptions mid-job — let them bubble to the runner's top-level handler.

```python
# In runner.py
try:
    job.run()
except Exception as e:
    logger.error(f"runner: job_failed error={e}")
    sys.exit(1)
```

---

## Common pitfalls

| Pitfall | What happens | Fix |
|---|---|---|
| `print()` instead of `logger` | Logs go nowhere useful; no timestamps; no rotation | Always use `from py_logging import get_logger` |
| `MERIDIAN_LOG_ROOT` not set | `EnvironmentError` at import | Set the env var before running; never catch and suppress |
| Not casting external string values | `'42.50' + 10 = '42.5010'` | Always cast: `float(row['amount'])` |
| Putting computation in `run()` | Untestable | Private `_methods` for all computation |
| Hardcoding DB credentials | Secrets in git | Always read from env vars; raise `EnvironmentError` if absent |
| No `finally` on DB connection | Connection leak under exceptions | Always `try/finally: client.close()` |
| Not using `pyproject.toml` | Bypasses uv | All modules use `pyproject.toml` with uv sources |
