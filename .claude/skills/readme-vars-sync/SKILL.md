---
name: readme-vars-sync
description: Check README.md's Role Variables table against defaults/main.yml and fix drift
---

`README.md`'s "Role Variables" section hand-mirrors `defaults/main.yml`. Whenever a default is added, removed, renamed, or its default value changes, the README can silently go stale.

Check:
1. Read `defaults/main.yml` — list every top-level variable and its default value/comment.
2. Read the "Role Variables" section of `README.md`.
3. Diff the two: variables in defaults but missing from README, variables in README but no longer in defaults, and default values that don't match.
4. Report the drift. If asked to fix it, edit the README table to match `defaults/main.yml` exactly — don't touch unrelated README sections.

Invoke this after any edit to `defaults/main.yml`, or on request before a release.
