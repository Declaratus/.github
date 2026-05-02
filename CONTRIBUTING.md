# Contributing to Declaratus

Thank you for your interest in contributing to Declaratus projects. This document describes
the contribution workflow shared across all `declaratus/*` repositories.

## Code of conduct

All participants are expected to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Reporting bugs and requesting features

Please open an issue in the relevant repository. Use the provided issue templates so that the
maintainers can triage and respond efficiently.

## Development setup

Each repository documents its own local development setup in its `README.md`. Follow those
instructions before making changes.

## Contribution workflow

1. **Fork** the repository and create a branch from `main`.
2. **Name your branch** using a short, descriptive slug, e.g. `fix/login-timeout` or
   `feat/export-csv`.
3. **Make your changes** in small, logical commits using Conventional Commits (see below).
4. **Push** your branch and open a pull request against `main`.
5. Address any review feedback. Once approved, a maintainer will **squash-merge** your PR.
   Your individual commits do not need to be perfectly structured; the squash commit message
   should follow Conventional Commits format.

## Conventional commits

All commit messages (and the final squash-merge commit title) must follow the
[Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<optional scope>): <short summary>

[optional body]

[optional footer(s)]
```

Allowed types:

| Type | When to use |
|------|-------------|
| `feat` | A new feature visible to users |
| `fix` | A bug fix |
| `docs` | Documentation changes only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `chore` | Build process, dependency updates, CI, tooling |
| `perf` | Performance improvements |
| `ci` | Changes to CI/CD configuration |

Breaking changes must include `BREAKING CHANGE:` in the commit footer, or append `!` after
the type: `feat!: remove deprecated endpoint`.

## Pull requests

- Fill in the [pull request template](.github/PULL_REQUEST_TEMPLATE.md) completely.
- Link the PR to any related issues using `Closes #<number>` or `Fixes #<number>`.
- Keep PRs focused; open separate PRs for unrelated changes.
- All CI checks must pass before a PR can be merged.

## Questions

If you have questions about using a Declaratus tool, please open a Discussion in the relevant
repository rather than filing an issue.
