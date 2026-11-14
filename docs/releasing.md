# Releasing

How every Declaratus repository versions, tags, writes its changelog and
publishes a release. A repository follows this once it ships versions: tags,
GitHub Releases, packages, images or binaries. Repositories that don't ship
versions, such as infrastructure, policy and docs repositories, don't need it
until they do.

## What is the same in every repository

| Part | Standard |
| --- | --- |
| Versions, tags, changelog | [release-please](https://github.com/googleapis/release-please) in manifest mode: `release-please-config.json` and `.release-please-manifest.json` at the root. One version for the whole repository, tagged `vX.Y.Z`. |
| Changelog | `CHANGELOG.md`, written by release-please. Don't edit generated entries by hand. |
| Commit messages | [Conventional Commits](https://www.conventionalcommits.org/) pull request titles, checked by `pr-title.yml`. |
| Merging | Squash merges only, with the pull request title as the commit message. The title is the changelog entry. |
| Token | One org GitHub App, used only by `release-please.yml`. |
| Workflows | Every action is pinned to a full commit SHA with a version comment. Dependabot keeps them current. |

### What changes between repositories

- `release-type` and `extra-files` in `release-please-config.json`: the
  files that carry the version, such as `pyproject.toml`, `Chart.yaml` or a
  plugin manifest.
- Whether there is anything to build. A repository that publishes builds
  (packages, binaries, images) has its own `.github/workflows/release.yml`,
  started by the release tag. One with nothing to build (schemas, skills,
  docs) publishes the GitHub Release straight from release-please.

## How a release happens

1. Pull requests merge to `main` with Conventional Commit titles.
2. On each push to `main`, `release-please.yml` opens or updates one release
   pull request, `chore(main): release X.Y.Z`. It bumps the version files,
   updates `.release-please-manifest.json`, and adds the next `CHANGELOG.md`
   entry. CI runs on it like any other pull request.
3. Merging the release pull request makes release-please create the `vX.Y.Z`
   tag and the GitHub Release, with the changelog entry as its notes.
4. In a repository that builds something, the release is a draft and the tag
   starts `release.yml`. That workflow builds from the tag, checks that the
   build reports the tag's version, attaches the outputs and publishes the
   draft. The release is only public once its files are attached.

`feat` bumps the minor version, `fix` and `perf` bump the patch, and `!`
after the type (`feat!: ...`) bumps the major. Before 1.0.0 a breaking change
bumps the minor (`bump-minor-pre-major`). `docs`, `ci`, `chore`, `test`,
`build`, `style` and `refactor` don't start a release by themselves and don't
appear in the changelog.

To set the next version by hand, set `release-as` in
`release-please-config.json` and remove it after that release, or add a
`Release-As: 1.0.0` footer to an entry in a pull request's
[`BEGIN_COMMIT_OVERRIDE` block](#pull-request-titles). The squash commit
carries only the title, so a footer anywhere else in the description is not
seen.

## Release candidates and prereleases

A release candidate is a tag that a maintainer pushes. It never goes through
release-please.

1. Pick the commit on `main` to test. Use the version the open release pull
   request proposes, or the next planned version if there isn't one yet.
2. Tag it and push the tag:

   ```sh
   git tag -a v0.1.0-rc.1 -m v0.1.0-rc.1 <commit>
   git push origin v0.1.0-rc.1
   ```

3. `release.yml` builds it with the tag's version, publishes it wherever a
   final release goes (a package registry treats it as a prerelease), and
   publishes the GitHub Release as a prerelease, since the tag has a suffix.
   Its notes are GitHub's generated notes since the previous tag. Nothing is
   committed for a candidate: see
   [where the version comes from](#where-the-version-comes-from).
4. Fix what the candidate turns up through ordinary pull requests, and tag
   `-rc.2` and so on as needed.
5. Release the final version by merging the release pull request.
   release-please reads its version from the manifest and ignores
   release-candidate tags, so the final changelog entry covers everything
   since the last final release.

Merging the release pull request adds a commit that changes only the files
release-please edits: the changelog, the manifest and the version files. If
other pull requests have merged since the candidate, they are in the release
too and listed in its changelog entry. Tag another candidate if they need
testing first.

release-please's own prerelease mode (`prerelease`, `versioning: prerelease`)
is not used. It turns the release pull request into a candidate pull request,
and leaving it means editing the config again.

## The `release.yml` contract

A repository that builds something has a `release.yml` that:

- runs on `push: tags: ['v*']`, so it handles both release-please tags and
  release-candidate tags;
- builds from the tag, takes the version from the tag, and fails if the build
  doesn't report it (see below);
- publishes to package registries with trusted publishing (OIDC) where the
  registry supports it, not stored tokens;
- ends with this publish step, which attaches the files to the draft
  release-please made or, for a release candidate, creates a prerelease:

```yaml
      - name: Publish the GitHub Release
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: |
          if gh release view "$TAG" --json isDraft >/dev/null 2>&1; then
            # release-please made a draft with the changelog entry as notes.
            gh release upload "$TAG" dist/* --clobber
            gh release edit "$TAG" --draft=false
          else
            # A tag pushed by hand, such as a release candidate.
            pre=""
            case "$TAG" in *-*) pre="--prerelease" ;; esac
            gh release create "$TAG" dist/* --verify-tag --title "$TAG" --generate-notes $pre
          fi
```

The job needs `permissions: contents: write`.

### Where the version comes from

The tag is the version. How a build gets it depends on where the project keeps
its version:

- **Read at build time** (Go ldflags, `git describe`, setuptools-scm): pass
  the tag in. Nothing else to do.
- **Kept in files** (`pyproject.toml`, `__version__`, `Chart.yaml`,
  `package.json`): release-please updates these files in the release pull
  request, so they match a final tag. `release.yml` checks that they do and
  fails if not. A release candidate is tagged on `main`, where the files still
  carry the last final version. For a tag with a suffix, `release.yml` writes
  the tag's version into those files in its own workspace before it builds,
  and never commits the change. Use the form each ecosystem expects:
  `v0.7.0-rc.1` is `0.7.0rc1` for Python (PEP 440) and stays `0.7.0-rc.1` for
  Helm charts, container tags and npm (SemVer).

Never bump version files by hand to cut a candidate. release-please owns them.

## Configuration

`release-please-config.json` for a repository that builds something:

```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json",
  "release-type": "simple",
  "include-component-in-tag": false,
  "bump-minor-pre-major": true,
  "draft": true,
  "force-tag-creation": true,
  "changelog-sections": [
    {"type": "feat", "section": "Features"},
    {"type": "fix", "section": "Bug Fixes"},
    {"type": "perf", "section": "Performance Improvements"},
    {"type": "revert", "section": "Reverts"},
    {"type": "deps", "section": "Dependencies"},
    {"type": "docs", "section": "Documentation", "hidden": true},
    {"type": "style", "section": "Styles", "hidden": true},
    {"type": "chore", "section": "Miscellaneous Chores", "hidden": true},
    {"type": "refactor", "section": "Code Refactoring", "hidden": true},
    {"type": "test", "section": "Tests", "hidden": true},
    {"type": "build", "section": "Build System", "hidden": true},
    {"type": "ci", "section": "Continuous Integration", "hidden": true}
  ],
  "packages": {
    ".": {
      "changelog-path": "CHANGELOG.md"
    }
  }
}
```

- `changelog-sections` goes in every config, unchanged. Some release types
  have their own defaults: `python` lists `docs` commits and lets a docs-only
  merge start a release. The list keeps every repository the same.
- `draft` and `force-tag-creation` go together. A draft release has no tag of
  its own, so `force-tag-creation` creates the tag first, and the tag starts
  `release.yml`. A repository with nothing to build leaves both out, and
  release-please publishes the release directly.
- `release-type`: `python` updates `pyproject.toml` and the package's
  `__version__`; `go` writes only the changelog, since Go reads the version
  from the tag; `node`, `helm` and `rust` update their manifests; `simple`
  keeps the version in `version.txt`.
- `extra-files`: any other file that carries the version. A file with a JSON
  path is `{"type": "json", "path": "...", "jsonpath": "$.version"}`; in
  other files, release-please updates lines marked with
  `x-release-please-version`.

`.release-please-manifest.json` holds the current version:

```json
{
  ".": "0.6.0"
}
```

For a repository that has never had a final release, use `{}` and set
`"initial-version": "0.1.0"` in the config; release-please otherwise starts
at 1.0.0. Set `"bootstrap-sha"` to the last commit on `main` before the
adoption pull request merges, so the first changelog entry doesn't list older
commits. release-please reads the commits after it, not the commit itself.

## Pull request titles

`<type>(<optional scope>)<optional !>: <summary>`, for example
`fix(stats): count rest days in the rolling rate` or
`feat!: read config from XDG paths only`. Allowed types: `feat`, `fix`,
`perf`, `revert`, `docs`, `ci`, `chore`, `test`, `build`, `style`,
`refactor`.

The summary is what readers of the changelog see. Write it for them: what
changed for someone using the project.

One pull request that carries several user-facing changes can list them in
its description, so each gets its own changelog line:

```text
BEGIN_COMMIT_OVERRIDE
feat: add the stats command

fix: keep worktrees on prune when the branch has unpushed commits
END_COMMIT_OVERRIDE
```

The block replaces the title as far as release-please is concerned. Leave a
blank line between entries. Mark a breaking entry with `!` after its type.

## Dependabot

Every adopting repository has a `.github/dependabot.yml` with a
`github-actions` entry. Its titles must pass the title check, so set the
prefix:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    commit-message:
      prefix: ci
      include: scope
```

Use `prefix: fix` with `include: scope` for the ecosystems that ship with the
project (`pip`, `uv`, `gomod`, `npm`), so a dependency update produces a
patch release. Use `prefix-development: chore` for development-only
dependencies.

## Adopting release-please in a repository

### Checklist

- [ ] The org release app is installed on the repository, and the repository
      can read `RELEASE_APP_CLIENT_ID` and `RELEASE_APP_PRIVATE_KEY` (see
      [One-time org setup](#one-time-org-setup)).
- [ ] Settings → General → Pull Requests: only **Allow squash merging** is
      on, with the default commit message set to **Pull request title**.
- [ ] Add `.github/workflows/release-please.yml` and
      `.github/workflows/pr-title.yml`. They are in this repository's
      [`workflow-templates/`](../workflow-templates), and on the
      **Actions → New workflow** page of any Declaratus repository.
- [ ] Add `release-please-config.json` and `.release-please-manifest.json`,
      with the current version in the manifest (`{}` if there is no final
      release yet).
- [ ] `release-type` and `extra-files` cover every file that carries the
      version, and `changelog-sections` is the standard list.
- [ ] If the repository builds something: `draft` and `force-tag-creation`
      in the config, and a `release.yml` that meets
      [the contract](#the-releaseyml-contract). Remove any step that checked
      out or created the release another way.
- [ ] `.github/dependabot.yml` keeps the actions current, with title
      prefixes that pass the check.
- [ ] A hand-written changelog is ready for generated entries (below).
- [ ] The contributing guide asks for Conventional Commit titles.
- [ ] The adoption pull request title is itself a Conventional Commit, such
      as `ci: release with release-please`.
- [ ] After merging, a `feat` or `fix` pull request makes release-please open
      a release pull request with the expected version.

### A repository with a hand-written changelog

release-please keeps what is already in `CHANGELOG.md`. It adds each new
entry above the first heading that looks like a version: `## 1.2.0`,
`## v1.2.0` or `## [1.2.0]`.

- Keep released entries as they are, including release-candidate entries.
  The heading style can differ from the generated one.
- Remove the `Unreleased` section. With `## [Unreleased]`, new entries land
  above it and it goes stale under them; with `## Unreleased`, it stays on top
  of every new entry.
- Carry anything listed under `Unreleased` into the first generated entry:
  put it in the adoption pull request's description as a
  `BEGIN_COMMIT_OVERRIDE` block, one Conventional Commit per change. The
  adoption pull request's own `ci:` line can go in the block too.
- Keep the file's opening lines if you like. release-please only adds its own
  `# Changelog` heading when the file has no version heading at all.
- If release notes were curated somewhere else, such as `docs/releases/`,
  stop adding new files there. The changelog entry is the release notes from
  now on.

## One-time org setup

These are org settings, made once by an org owner.

1. **The release app.** Create a GitHub App owned by the org, with no
   webhook, and these repository permissions: Contents read and write, Pull
   requests read and write, Issues read and write (release-please labels
   the release pull request), and Metadata read. Install it on only the
   repositories that adopt this standard.
2. **Its credentials.** Add an org variable `RELEASE_APP_CLIENT_ID` (the
   app's client ID) and an org secret `RELEASE_APP_PRIVATE_KEY` (a private
   key generated for the app). Give both the same repository access as the
   installation. Never commit either.
3. **Merge settings.** Each adopting repository allows squash merges only,
   with the pull request title as the commit message (see the checklist).
   Rulesets can require squash merges but can't set the commit message, so
   this stays a repository setting.
4. **Rulesets.** These are managed as code in `Declaratus/governance`:
   - `allowed_merge_methods: ["squash"]` in the `pull_request` rule for
     adopting repositories;
   - the `conventional-commits` check from `pr-title.yml` as a required
     status check for adopting repositories;
   - optionally, a tag ruleset for `refs/tags/v*` that restricts creation,
     with the release app, org admins and the repository's maintainers as
     bypass actors. Maintainers need the bypass to push release-candidate
     tags.

   Target these at adopting repositories, for example through a custom
   property such as `release-process: release-please`. A required check
   would block every pull request in a repository that doesn't have
   `pr-title.yml`. Start each ruleset in `evaluate`, as the other governance
   rulesets did.

Branch rules that ask for signed commits or linear history don't get in the
way. Squash merges from the web interface are signed by GitHub, and
release-please never pushes to `main` itself.

## Why the workflows are copied, not called

`release-please.yml` and `pr-title.yml` are short, and they are the same in
every repository, so each repository keeps a copy from
[`workflow-templates/`](../workflow-templates). A reusable workflow called by
SHA would need this repository to publish tagged releases of its own before
Dependabot could update callers, and it would put the release app's
credentials one more hop away from the repository that uses them. A copy has
its action SHAs updated by that repository's Dependabot. When the template
changes, update the copies with a `ci:` pull request in each repository.
