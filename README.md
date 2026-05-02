# .github

Organization-wide community health files and reusable GitHub Actions workflows for the
Declaratus organization.

## Reusable workflows

### `ci-python.yml`

Reusable Python CI workflow. Runs lint (ruff), type checking (mypy), and tests (pytest)
in a matrix across Python versions and operating systems.

**Caller prerequisites:**

- The project must have a `pyproject.toml` or `setup.cfg` with pip extras defined
  (default extra: `dev`). The `dev` extra (or whichever extra is passed via the `extras`
  input) must include all test and type-checking dependencies.
- `pytest-cov` must be listed as a dependency (typically under the `dev` extra) so that
  `pytest --cov --cov-report=xml` works without additional installation steps.
- `mypy` should be listed as a dependency under the configured extras so that type
  information for all project imports is available during type checking. The workflow
  also installs `mypy` explicitly as a fallback, but project-level stubs and plugins
  must be declared in the project dependencies.

**Example caller workflow:**

```yaml
jobs:
  ci:
    uses: declaratus/.github/.github/workflows/ci-python.yml@main
    with:
      python-versions: '["3.11", "3.12", "3.13"]'
      extras: dev
```

### `release-pypi.yml`

Reusable PyPI release workflow using OIDC trusted publishing (no API tokens required).

**Caller prerequisites:**

- Configure a trusted publisher on PyPI for the calling repository:
  - Publisher: GitHub Actions
  - Repository owner: your GitHub org/user
  - Repository name: your repo name
  - Workflow filename: the caller workflow file name (e.g. `release.yml`)
  - Environment name: `pypi`
- Create a GitHub Actions environment named `pypi` in the calling repository
  (Settings > Environments). Add any required reviewers or deployment protection rules.

**Example caller workflow:**

```yaml
jobs:
  release:
    uses: declaratus/.github/.github/workflows/release-pypi.yml@main
    with:
      tag: ${{ github.ref_name }}
```

### `codeql.yml`

Reusable CodeQL security analysis workflow.

**Example caller workflow:**

```yaml
jobs:
  codeql:
    uses: declaratus/.github/.github/workflows/codeql.yml@main
    with:
      languages: '["python"]'
```

## Community health files

The following files are automatically inherited by all `declaratus/*` repositories:

- `CONTRIBUTING.md` — contribution workflow
- `CODE_OF_CONDUCT.md` — Contributor Covenant 2.1
- `SECURITY.md` — vulnerability reporting policy
- `SUPPORT.md` — where to get help
