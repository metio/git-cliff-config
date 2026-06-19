<!--
SPDX-FileCopyrightText: The git-cliff-config Authors
SPDX-License-Identifier: 0BSD
-->

# git-cliff-config

Org-wide [git-cliff](https://git-cliff.org/) configuration for the metio
projects, so the changelog / release-note convention lives in exactly one place
(the same idea as [renovate-config](https://github.com/metio/renovate-config) and
[vale-config](https://github.com/metio/vale-config)).

## Files

- **`cliff.toml`** — release notes / changelog. Serves both single-project repos
  (jaas, stageset-controller) and the helm-charts monorepo.
- **`cliff-artifacthub.toml`** — renders the same commit range as an Artifact Hub
  `artifacthub.io/changes` annotation (used only by helm-charts). Its `[git]`
  section mirrors `cliff.toml`; keeping both here keeps them in sync.

## How a repo consumes it

git-cliff loads a remote config with `--config-url`. Each release workflow pins
the URL to a commit (bumped by Renovate, like an image digest):

```sh
git-cliff \
  --config-url "https://raw.githubusercontent.com/metio/git-cliff-config/<commit-sha>/cliff.toml" \
  --tag "<version>" <prev>..HEAD > notes.md
```

The monorepo additionally passes `--include-path charts/<name>/**` and
`--tag-pattern ^<name>-`, and sets `CHART` / `VERSION` so the heading carries the
chart name. `GITHUB_TOKEN` must be in the environment so git-cliff can enrich
entries with pull-request numbers and contributors.

A Renovate custom manager in each repo keeps the pinned commit current:

```jsonc
"customManagers": [{
  "customType": "regex",
  "managerFilePatterns": ["/^\\.github/workflows/release\\.yml$/"],
  "matchStrings": ["git-cliff-config/(?<currentDigest>[a-f0-9]{40})/"],
  "currentValueTemplate": "main",
  "depNameTemplate": "metio/git-cliff-config",
  "packageNameTemplate": "https://github.com/metio/git-cliff-config",
  "datasourceTemplate": "git-refs"
}]
```

## Commit convention

Commits follow [Conventional Commits](https://www.conventionalcommits.org/). The
type maps to a changelog group: `feat` → Added, `fix` → Fixed, `doc` →
Documentation, `perf` → Performance, `refactor`/`chore` → Changed,
`chore(deps)` → Dependencies, `revert` → Reverted.

**Breaking changes** use a `!` after the type (`feat(api)!: …`) or a
`BREAKING CHANGE:` footer. They render in a dedicated **⚠ Breaking changes**
section; the operator-facing upgrade steps belong in each project's
`MIGRATIONS.md`.
