---
name: changelog-from-tags
description: Generate release notes from git log between the last tag and HEAD
disable-model-invocation: true
---

Releases are tag-driven (see `release-role` skill). This generates a paste-ready GitHub Release body — it doesn't replace maintaining `CHANGELOG.md` by hand, only saves re-deriving it from commits:

1. Find the last tag: `git tag --sort=-v:refname | head -1`
2. `git log <last-tag>..HEAD --pretty=format:'%s (%h)'`
3. Group the commit subjects into rough buckets by prefix/keyword: Fixes, Features/Additions, Docs/CI, Other. Don't over-engineer the categorization — this is a small role, a flat list under 1-2 headers is fine if there aren't many commits.
4. Output as Markdown, ready to paste into a GitHub Release description.

If there's no previous tag, use the full `git log --pretty=format:'%s (%h)'` history instead.
