# Changelog

All notable changes to this project are documented in this file.

## [Unreleased]
- Fix: the `sftp` subsystem used the Debian path on every OS, which does not exist on the RedHat family. The path and the service name now come from `vars/<os_family>.yml`.
- Fix: the role works with `gather_facts: false`. Argument validation no longer needs facts, and the role gathers the `min` subset itself.
- Change: the rendered configs use `{{ ansible_managed | comment }}`.
- Change: `min_ansible_version` is 2.16, which matches what CI tests.
- Change: the summary message only shows at `-v`.
- Removed: `ssh_protocol`, `ssh_client_protocol` and `ssh_use_privilege_separation`. No template used them, and OpenSSH dropped `Protocol` and `UsePrivilegeSeparation`.
- Removed: the `validate ssh config` and `restart ssh with validation` handlers, which nothing notified.
- CI: lint runs the pinned pre-commit hooks (now with gitleaks), actions are pinned by SHA, and Molecule uses the Podman plugin it actually runs.

## [1.0.1] - 2025-10-28
- Simplified the ssh restart validation handler.

## [1.0.0] - 2025-10-27
- Initial release
- Comprehensive SSH hardening implementation
- Support for major Linux distributions
- Security compliance features (aligned with CIS/NIST guidance)

## Format
This changelog follows a simple, readable format. Future versions may adopt the
Keep a Changelog style and semantic versioning.