---
name: release-role
description: Pre-flight checks then tag and push a new release for this Ansible role (Galaxy publishes on tag push)
disable-model-invocation: true
---

This role has no version field to bump — release = push a `vX.Y.Z` tag, which triggers `.github/workflows/molecule.yml`. Its `release` job (gated by `needs: [lint, molecule]` and `startsWith(github.ref, 'refs/tags/v')`) publishes to Galaxy after lint + the full distro matrix pass.

Steps:

1. **Pre-flight checks** — stop and report if any fail:
   - `git status --porcelain` is empty (no uncommitted changes)
   - Current branch is `main` and up to date with `origin/main` (`git fetch && git status`)
   - Determine the next version: `git tag --sort=-v:refname | head -1` for the last tag, ask the user to confirm the bump (major/minor/patch) unless they already specified one
   - Confirm the target tag doesn't already exist: `git tag -l vX.Y.Z`

2. **Tag and push** (confirm with the user before pushing — this is irreversible and triggers CI + a public Galaxy release):
   ```bash
   git tag -a vX.Y.Z -m "vX.Y.Z"
   git push origin vX.Y.Z
   ```

3. **Verify**: point the user at the Actions run (`gh run list --workflow=molecule.yml --limit 1`) so they can confirm the `lint` and `molecule` jobs pass before the `release` job runs.

Never force-push tags or delete/re-push an existing tag without explicit confirmation — Galaxy may have already indexed it.
