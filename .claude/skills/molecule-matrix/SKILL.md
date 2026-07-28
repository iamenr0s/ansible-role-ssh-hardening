---
name: molecule-matrix
description: Run molecule test against every distro in the CI matrix and report pass/fail per distro
disable-model-invocation: true
---

Run `molecule test` once per distro from the CI matrix (`.github/workflows/molecule.yml`), instead of the user manually re-running `MOLECULE_DISTRO=x molecule test` for each one.

Distros (keep in sync with the `strategy.matrix.distro` list in `.github/workflows/molecule.yml`):
almalinux8 almalinux9 almalinux10 debian12 debian13 fedora42 fedora43 fedora44 rockylinux8 rockylinux9 rockylinux10 ubuntu2204 ubuntu2404

Run:

```bash
declare -A results
for distro in almalinux8 almalinux9 almalinux10 debian12 debian13 \
              fedora42 fedora43 fedora44 rockylinux8 rockylinux9 rockylinux10 \
              ubuntu2204 ubuntu2404; do
  echo "=== $distro ==="
  if MOLECULE_DISTRO="$distro" molecule test; then
    results[$distro]="PASS"
  else
    results[$distro]="FAIL"
  fi
done

echo
echo "=== Summary ==="
for distro in "${!results[@]}"; do
  printf "%-15s %s\n" "$distro" "${results[$distro]}"
done
```

Report the summary table to the user. If a distro fails, don't debug all of them inline — surface which ones failed and let the user pick which to investigate (`MOLECULE_DISTRO=<distro> molecule test` reruns just that one).

Requires Docker/Podman and the `molecule`, `molecule-plugins[docker]`, `ansible` Python packages (see project `CLAUDE.md`).
