# quick-venv

[![CI](https://github.com/tokenoodle-everything/quick-venv/actions/workflows/ci.yml/badge.svg)](https://github.com/tokenoodle-everything/quick-venv/actions/workflows/ci.yml)

A GitHub Action that gives you a ready-to-use Python virtual environment in
seconds, powered by [tn-venv](https://github.com/tokenoodle-everything/tn-venv).

Use it once, and every subsequent step in your job automatically runs inside
the virtual environment — `python` and `pip` resolve into it, no activation
boilerplate, no manual caching. It's fast.

## Usage

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: tokenoodle-everything/quick-venv@v1
    with:
      python-version: "3.12"

  # From here on, `python` and `pip` run inside the venv.
  - run: python -m pip install -r requirements-dev.txt
  - run: python -m pytest
```

By default the action:

1. provisions the requested Python via `actions/setup-python`,
2. installs `tn-venv` from PyPI,
3. restores the environment from cache when possible, otherwise creates it
   with `tn-venv` (and refreshes a cache-restored one in place with
   `--upgrade`, which is effectively instant),
4. puts the environment's `bin`/`Scripts` directory on `PATH` and exports
   `VIRTUAL_ENV` for all later steps,
5. installs your `requirements.txt` (if present) into the environment.

The environment is cached across runs, keyed on OS, architecture, Python
version, and a hash of your dependency files — so repeat runs typically cost
a couple of seconds.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `python-version` | `"3.x"` | Python version to build the environment from (via `actions/setup-python`). |
| `dest` | `".venv"` | Where the environment is created (relative to the workspace root). |
| `requirements` | `"requirements.txt"` | Whitespace-separated requirements files to pip-install. Missing files are skipped. |
| `install` | `"true"` | Set to `"false"` to skip installing `requirements`. |
| `cache` | `"true"` | Set to `"false"` to disable caching of the environment. |
| `tn-venv-install` | `"tn-venv"` | pip spec for tn-venv itself, e.g. `"tn-venv==0.1.1"` or a `git+https://…` URL. |
| `tn-venv-args` | `""` | Extra arguments for tn-venv, e.g. `"--system-site-packages --upgrade-pip"`. |

## Outputs

| Output | Description |
| --- | --- |
| `python-path` | Absolute path to the environment's Python interpreter. |
| `venv-path` | Absolute path to the virtual environment directory. |
| `cache-hit` | Whether the environment was restored from cache (`"true"` / `"false"`). |

## Example: full control

```yaml
- uses: tokenoodle-everything/quick-venv@v1
  id: venv
  with:
    python-version: "3.13"
    dest: ".venv"
    requirements: requirements.txt requirements-dev.txt
    tn-venv-install: "git+https://github.com/tokenoodle-everything/tn-venv@master"
    tn-venv-args: "--upgrade-pip"

- run: ${{ steps.venv.outputs.python-path }} -m pytest
```

## How it stays fast

- `tn-venv` creates environments in a fraction of the time a full
  `python -m venv` + `pip` bootstrap takes.
- The whole environment directory is cached; on a hit, `tn-venv --upgrade`
  only reconciles it with the current interpreter instead of rebuilding.
- Dependency installation runs after the cache restore, so a warm cache with
  unchanged requirements does essentially no work.

## License

See [License.txt](License.txt).
