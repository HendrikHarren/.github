# HendrikHarren/.github — Shared Reusable Workflows

Central repository for GitHub Actions reusable workflows used across all HendrikHarren repositories.

## Available Workflows

| Workflow | Description | Timeout |
|---|---|---|
| `reusable-tests.yml` | Python test suite with coverage | 15 min |
| `reusable-type-check.yml` | mypy type checking with baseline | 15 min |
| `reusable-security.yml` | pip-audit + bandit + TruffleHog (pinned) | 10 min |
| `reusable-claude-review.yml` | Claude Code AI code review | 20 min |

All workflows include: **job timeouts**, **TruffleHog pinned to `v3.93.3`**.

Callers are responsible for: **path filters** and **concurrency groups** (project-specific).

## Usage

In your project's `.github/workflows/tests.yml`:

```yaml
name: Tests

on:
  push:
    branches: [main]
    paths: [src/**, tests/**, requirements*.txt]
  pull_request:
    branches: [main]
    paths: [src/**, tests/**, requirements*.txt]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    uses: HendrikHarren/.github/.github/workflows/reusable-tests.yml@main
    with:
      python-version: '3.12'
      pytest-args: 'tests/unit/ -m "not slow" --cov=src --cov-report=html --cov-report=term-missing'
```

## Inputs Reference

### `reusable-tests.yml`
| Input | Default | Description |
|---|---|---|
| `python-version` | `3.12` | Python version |
| `pytest-args` | `tests/unit/ -m "not slow" --cov=src ...` | pytest arguments |
| `requirements-file` | `requirements-dev.txt` | Requirements file |

### `reusable-type-check.yml`
| Input | Default | Description |
|---|---|---|
| `python-version` | `3.12` | Python version |
| `mypy-baseline` | `0` | Max allowed mypy errors |
| `requirements-files` | `requirements.txt requirements-dev.txt` | Space-separated files |

### `reusable-security.yml`
| Input | Default | Description |
|---|---|---|
| `python-version` | `3.12` | Python version |
| `requirements-files` | `requirements.txt requirements-dev.txt` | Space-separated files |
| `bandit-target` | `src/` | Directory to scan |
| `notify-issue-assignee` | `` | GitHub username to @mention on failure |

### `reusable-claude-review.yml`
| Input | Default | Description |
|---|---|---|
| `model` | `claude-sonnet-4-6` | Claude model ID |
| `effort-level` | `medium` | `low`, `medium`, or `high` |
| `review-prompt` | _(default prompt)_ | Custom review instructions |

**Secret required:** `CLAUDE_CODE_OAUTH_TOKEN`

## Updating TruffleHog Version

When a new TruffleHog release is available, update the pinned version in `reusable-security.yml`:
```yaml
uses: trufflesecurity/trufflehog@v3.XX.X  # update here
```
All repos using this workflow inherit the update automatically.
