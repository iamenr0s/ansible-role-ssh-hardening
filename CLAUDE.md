# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Linting
```bash
# YAML lint (matches CI)
yamllint .

# Ansible lint
ansible-lint
```

### Molecule testing (requires Docker/Podman)
```bash
# Install test dependencies (ansible<10 keeps ansible-core 2.16.x on the
# EL8 legs — newer ansible-core requires Python 3.8+ on managed nodes,
# which breaks AlmaLinux 8/RockyLinux 8's Python 3.6.8)
pip3 install ansible molecule molecule-plugins[docker] docker cryptography

# Run tests against default distro (almalinux9)
molecule test

# Run tests against a specific distro
MOLECULE_DISTRO=ubuntu2404 molecule test
MOLECULE_DISTRO=debian13 molecule test
MOLECULE_DISTRO=fedora44 molecule test

# Run individual molecule phases
molecule create
molecule converge
molecule verify
molecule destroy
```

### Install Ansible collection dependencies
```bash
ansible-galaxy collection install -r requirements.yml
```
(Currently a no-op — the role only uses `ansible.builtin` modules.)

## Architecture

This is an Ansible role (`iamenr0s.ansible_role_ssh_hardening`) that hardens the OpenSSH daemon, and optionally the system-wide SSH client, across Debian/Ubuntu and RHEL-family (Alma/Rocky/Fedora) hosts.

### Execution flow

`tasks/main.yml` is a single, linear task file (no per-OS dispatch — the only OS-family branch is `ssh_service_name` in `defaults/main.yml`, via `ansible_facts['os_family']`; the package name `openssh-server` is identical across Debian and RHEL family, so `ssh_package_name` is a plain constant, not templated):

1. Install `ssh_package_name`.
2. Generate SSH host keys if missing (`ssh-keygen -A`), notifying `restart ssh`.
3. Fix ownership/permissions on host key and public key files.
4. Render the hardened `sshd_config` from `templates/sshd_config.j2`, validated with `sshd -t -f %s` via the template task's `validate` option before it's ever written live, with `backup: yes` so the pre-existing config is preserved (only when it actually changes — this is what keeps the role idempotent); notifies `restart ssh`.
5. Deploy the pre-login banner (`templates/ssh_banner.j2`) when `ssh_banner_enabled`; notifies `reload ssh`.
6. Remove weak DSA host keys; notifies `restart ssh`.
7. Render the client config (`templates/ssh_config.j2`) to `/etc/ssh/ssh_config` when `configure_ssh_client` (also `backup: yes`).
8. Ensure the SSH service is started and enabled.
9. Validate the live config with `sshd -t`.
10. Print a summary of the applied hardening settings.

Note: there is deliberately no separate up-front "backup original config" task — an earlier version had one that copied to a filename containing `ansible_date_time.epoch`, which made it non-idempotent (a new file every run, unconditionally "changed"). The `backup: yes` on the template tasks above covers the same need and only fires on a real change.

### Handler wiring

`handlers/main.yml` defines: `restart ssh` (listens `"restart ssh"`), `reload ssh` (listens `"reload ssh"`), and a validated pair `validate ssh config`/`restart ssh with validation` for opt-in use in custom task additions. `notify:` names must match a handler's `listen:`/`name:` exactly, or the handler silently never fires — see the `handler-notify-check` skill.

### Testing

Molecule uses Podman locally (`driver: podman`) but Docker in CI. The `MOLECULE_DISTRO` env var selects the container image from `iamenr0s/docker-<distro>-ansible:latest`. `converge.yml` bootstraps Python 3 via `raw` (some base images lack it), resets the connection, then runs an explicit `ansible.builtin.setup` — needed because this role's defaults (`ansible_facts['os_family']`) read facts, and the converge play itself runs with `gather_facts: false`. `verify.yml` asserts the rendered `sshd_config`/`ssh_config` contain the expected hardening directives, that DSA host keys are gone, that the SSH service is running, and that the banner file exists when enabled. The `idempotence` step in molecule's default test_sequence reruns converge and fails the build if any task reports `changed` the second time — this is what caught the old non-idempotent backup task.

### Lint rules

`.yamllint` extends `default` with `indentation: disable`, `line-length: disable`, `trailing-spaces: disable`, and `truthy: disable`. `.ansible-lint` skips rules `106` and `503` — see the `ansible-lint-exceptions-audit` skill before assuming that's still warranted.

### CI triggers

- **molecule.yml**: Runs lint + full matrix test on pushes to `main`, tags (`v*.*.*`), and PRs. On `v*` tags, a `release` job (gated by `needs: [lint, molecule]`) publishes to Galaxy.
- **code-scanning-notify.yml**: Polls the code-scanning API on a 6-hour schedule (`code_scanning_alert` is webhook-only, not a valid Actions trigger) and posts new/updated open alerts to the webhook in the `SECURITY_ALERT_WEBHOOK` secret (Slack format; Discord works by appending `/slack` to the webhook URL).
