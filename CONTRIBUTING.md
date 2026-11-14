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
3. **Make your changes** in small, logical commits.
4. **Push** your branch and open a pull request against `main`, with a Conventional Commit
   title (see below).
5. Address any review feedback. Once approved, a maintainer will **squash-merge** your PR.
   The pull request title becomes the squash commit message and, in repositories that
   release, the changelog entry. Your individual commits don't need any particular format.

## Pull request titles

Pull request titles follow the [Conventional Commits](https://www.conventionalcommits.org/)
specification. A `pr-title` check enforces this in repositories that release:

```
<type>(<optional scope>)<optional !>: <short summary>
```

Allowed types:

| Type | When to use |
|------|-------------|
| `feat` | A new feature visible to users |
| `fix` | A bug fix |
| `docs` | Documentation changes only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `perf` | Performance improvements |
| `revert` | Reverting an earlier change |
| `build` | Build system or packaging |
| `style` | Formatting with no code change |
| `ci` | Changes to CI/CD configuration |
| `chore` | Maintenance and tooling |

`feat`, `fix`, `perf` and `revert` appear in the changelog and start a release. The other
types don't. Mark a breaking change with `!` after the type, as in
`feat!: remove deprecated endpoint`. The squash commit carries only the title, so a
`BREAKING CHANGE:` footer in the description is not seen. To list several changes from one
pull request, see [Pull request titles](docs/releasing.md#pull-request-titles) in the release
standard.

## Pull requests

- Fill in the [pull request template](.github/PULL_REQUEST_TEMPLATE.md) completely.
- Link the PR to any related issues using `Closes #<number>` or `Fixes #<number>`.
- Keep PRs focused; open separate PRs for unrelated changes.
- All CI checks must pass before a PR can be merged.

## Questions

If you have questions about using a Declaratus tool, please open a Discussion in the relevant
repository rather than filing an issue.
