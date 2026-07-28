---
name: distro-support-bump
description: Add or drop a supported OS/distro version consistently across all 3 places this role tracks it
disable-model-invocation: true
---

Supported-OS info for this role is duplicated across three files. When adding/dropping a distro version (e.g. "add Fedora 45", "drop Ubuntu 20"), update all three or the sources drift:

1. **`meta/main.yml`** — `galaxy_info.platforms`, Galaxy's own format: grouped by platform name (`EL`, `Fedora`, `Debian`, `Ubuntu`, etc.) with a `versions` list. Debian/Ubuntu use codenames (`bookworm`, `jammy`), not version numbers — ansible-lint's `schema[meta]` rule rejects `"12"`/`"22.04"` for those families.

2. **`.github/workflows/molecule.yml`** — `jobs.molecule.strategy.matrix.distro` list, using Docker image tag naming (e.g. `fedora43`, `rockylinux9`, `ubuntu2404` — no spaces, RHEL-family uses `almalinux`/`rockylinux` prefix). Also confirm an `iamenr0s/docker-<distro>-ansible:latest` image exists for a new distro before adding it — the matrix will hang/fail otherwise. EL8 legs (`almalinux8`, `rockylinux8`) need an `include:` entry pinning `ansible_pin: "ansible<10"` and `python_version: '3.12'`.

3. **`README.md`** — "Supported Platforms" bullet list, matching the same distro set as the workflow matrix.

Also check `.claude/skills/molecule-matrix/SKILL.md`'s distro list — it's a plain copy of #2 and needs the same edit.

Note: this role has no `defaults/main.yml` runtime OS-support guard (unlike roles with an `*_stable_os` pre-check) — supported platforms here are advisory (Galaxy metadata + CI coverage), not enforced at runtime.

After editing, show the user a diff summary across all three (four, counting the molecule-matrix skill) files before considering it done — this is a "did I get all the places" task, not a single-file edit.
