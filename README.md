[![Molecule](https://github.com/iamenr0s/ansible-role-ssh-hardening/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-ssh-hardening/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_ssh_hardening) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-ssh-hardening/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-ssh-hardening)

Ansible Role: SSH Hardening
============================

A comprehensive Ansible role for hardening the OpenSSH daemon (and optionally the system-wide SSH client) with security best practices. It renders a hardened `sshd_config` from strong cryptographic defaults, removes weak host keys, deploys a pre-login banner, and validates the configuration before restarting the service.

Features
--------
- Strong cryptography only: curated cipher, MAC, and key exchange algorithm lists.
- Authentication hardening: root login, password auth, empty passwords, and Kerberos/GSSAPI all disabled by default; public key auth only.
- Access control via `ssh_allow_users`/`ssh_deny_users`/`ssh_allow_groups`/`ssh_deny_groups` and `Match` blocks.
- Removes weak DSA host keys and regenerates missing host keys (`ssh-keygen -A`).
- Deploys a configurable pre-login banner.
- Validates the rendered config with `sshd -t` before and after restart (`validate` on the template task, plus a `validate ssh config` handler).
- Backs up the original `sshd_config` before overwriting it.
- Optionally hardens the system-wide SSH client (`/etc/ssh/ssh_config`).

Requirements
------------
- Ansible 2.9+.
- Target systems with OpenSSH installed (or installable via the OS package manager — the role installs `ssh_package_name` if missing).
- Run with privilege escalation on real hosts: `become: true` is recommended.
- No non-`ansible.builtin` collections are required.

Supported Platforms
--------------------
- AlmaLinux 8, 9, 10
- RockyLinux 8, 9, 10
- Fedora 42, 43, 44
- Debian 12 (bookworm), 13 (trixie)
- Ubuntu 22.04 (jammy), 24.04 (noble)

Note: Platforms in `meta/main.yml` reflect Galaxy metadata (AlmaLinux/RockyLinux are represented under the generic `EL` platform there); the list above matches the distros actually exercised by the Molecule CI matrix.

Role Variables
--------------
All variables are defined in `defaults/main.yml`, grouped below in the same order.

### Package and service

- `ssh_package_name` (str): SSH server package to install (default: `openssh-server` — same package name across Debian and RHEL family).
- `ssh_service_name` (str): SSH service to manage (default: `ssh` on Debian family, `sshd` elsewhere).
- `ssh_config_file` (str): Path to the rendered sshd config (default: `/etc/ssh/sshd_config`).

### Network

- `ssh_port` (int): Port sshd listens on (default: `22`).
- `ssh_listen_addresses` (list): Addresses sshd listens on; empty means all interfaces (default: `[]`).
- `ssh_address_family` (str): `any`, `inet`, or `inet6` (default: `any`).

### Protocol

- `ssh_protocol` (int): SSH protocol version (default: `2`).

### Host key types

- `ssh_host_key_types` (list): Host key algorithms to use (default: `[rsa, ecdsa, ed25519]`).

### Cryptographic algorithms

- `ssh_kex_algorithms` (list): Allowed key exchange algorithms (default: curve25519/DH-group16/18/14 SHA2/SHA512 set — see `defaults/main.yml`).
- `ssh_ciphers` (list): Allowed ciphers (default: chacha20-poly1305 + AES-GCM/CTR set).
- `ssh_macs` (list): Allowed MAC algorithms (default: SHA-2 ETM + non-ETM set).

### Authentication

- `ssh_login_grace_time` (int): Seconds allowed to complete authentication (default: `30`).
- `ssh_permit_root_login` (str): Root login policy (default: `"no"`).
- `ssh_strict_modes` (str): Enforce file mode/ownership checks (default: `"yes"`).
- `ssh_max_auth_tries` (int): Max authentication attempts per connection (default: `3`).
- `ssh_max_sessions` (int): Max open sessions per connection (default: `2`).
- `ssh_max_startups` (str): Unauthenticated connection throttling, `start:rate:full` (default: `"10:30:60"`).

### Public key authentication

- `ssh_pubkey_authentication` (str): Enable public key auth (default: `"yes"`).
- `ssh_authorized_keys_file` (str): Path(s) to authorized_keys, relative to home (default: `.ssh/authorized_keys`).

### Password authentication

- `ssh_password_authentication` (str): Enable password auth (default: `"no"`).
- `ssh_permit_empty_passwords` (str): Allow empty passwords (default: `"no"`).
- `ssh_challenge_response_auth` (str): Enable challenge-response/keyboard-interactive auth (default: `"no"`).

### Kerberos authentication

- `ssh_kerberos_authentication` (str): Enable Kerberos auth (default: `"no"`).
- `ssh_kerberos_or_local_passwd` (str): Fall back to local password auth (default: `"no"`).
- `ssh_kerberos_ticket_cleanup` (str): Destroy ticket cache on logout (default: `"yes"`).

### GSSAPI authentication

- `ssh_gssapi_authentication` (str): Enable GSSAPI auth (default: `"no"`).
- `ssh_gssapi_cleanup_credentials` (str): Destroy GSSAPI credentials on logout (default: `"yes"`).

### User and group restrictions

- `ssh_allow_users` (list): Usernames allowed to log in (default: `[]`, no restriction).
- `ssh_deny_users` (list): Usernames denied (default: `[]`).
- `ssh_allow_groups` (list): Groups allowed (default: `[]`, no restriction).
- `ssh_deny_groups` (list): Groups denied (default: `[]`).

### Forwarding and tunneling

- `ssh_allow_agent_forwarding` (str): Allow ssh-agent forwarding (default: `"no"`).
- `ssh_allow_tcp_forwarding` (str): Allow TCP forwarding (default: `"no"`).
- `ssh_gateway_ports` (str): Allow remote hosts to connect to forwarded ports (default: `"no"`).
- `ssh_x11_forwarding` (str): Allow X11 forwarding (default: `"no"`).
- `ssh_x11_display_offset` (int): First display number for X11 forwarding (default: `10`).
- `ssh_x11_use_localhost` (str): Bind X11 forwarding to loopback (default: `"yes"`).
- `ssh_permit_tty` (str): Allow pty allocation (default: `"yes"`).
- `ssh_permit_tunnel` (str): Allow tun device forwarding (default: `"no"`).

### Environment

- `ssh_permit_user_environment` (str): Process user environment files (default: `"no"`).
- `ssh_accept_env` (list): Environment variable patterns accepted from the client (default: `[LANG, "LC_*"]`).

### Compression

- `ssh_compression` (str): Enable compression (default: `"no"`).

### Client alive settings

- `ssh_client_alive_interval` (int): Seconds between server keepalives to idle clients (default: `300`).
- `ssh_client_alive_count_max` (int): Unanswered keepalives before disconnect (default: `2`).

### Logging

- `ssh_syslog_facility` (str): Syslog facility (default: `AUTH`).
- `ssh_log_level` (str): Logging verbosity (default: `INFO`).

### Banner

- `ssh_banner_enabled` (bool): Deploy and configure a pre-login banner (default: `true`).
- `ssh_banner_file` (str): Path the banner is deployed to (default: `/etc/ssh/banner`).
- `ssh_banner_content` (str): Banner file contents (default: an "AUTHORIZED ACCESS ONLY" notice — see `defaults/main.yml`).

### Subsystems

- `ssh_subsystems` (list of dicts): Subsystems to configure, each with `name` and `command` (default: `[{name: sftp, command: /usr/lib/openssh/sftp-server}]`).

### Additional security settings

- `ssh_use_dns` (str): Resolve client hostnames for access control/logging (default: `"no"`).
- `ssh_permit_user_rc` (str): Execute `~/.ssh/rc` on login (default: `"no"`).
- `ssh_print_motd` (str): Print `/etc/motd` on login (default: `"no"`).
- `ssh_print_last_log` (str): Print last login time (default: `"yes"`).
- `ssh_tcp_keep_alive` (str): Send TCP keepalives (default: `"yes"`).
- `ssh_use_privilege_separation` (str): Privilege separation mode (default: `"yes"`).

### Authentication methods

- `ssh_authentication_methods` (list): Required authentication method combination(s) (default: `[publickey]`).

### Trusted CA keys

- `ssh_trusted_user_ca_keys` (str): Path to a CA public keys file trusted for user certificates; empty disables (default: `""`).

### Match blocks

- `ssh_match_blocks` (list of dicts): Conditional configuration blocks, each with a `condition` (the `Match` criteria string) and a `settings` dict of directive/value pairs (default: `[]`). Example:
  ```yaml
  ssh_match_blocks:
    - condition: "User admin"
      settings:
        PasswordAuthentication: "yes"
        MaxAuthTries: "5"
  ```

### SSH client configuration

- `configure_ssh_client` (bool): Manage the system-wide SSH client config (default: `true`).
- `ssh_client_protocol` (int): Client SSH protocol version (default: `2`).
- `ssh_client_kex_algorithms` (list): Client key exchange algorithms (default: `ssh_kex_algorithms`).
- `ssh_client_ciphers` (list): Client ciphers (default: `ssh_ciphers`).
- `ssh_client_macs` (list): Client MAC algorithms (default: `ssh_macs`).
- `ssh_client_host_key_algorithms` (list): Host key algorithms the client accepts (default: ed25519/rsa cert + plain ed25519/ecdsa/rsa set — see `defaults/main.yml`).
- `ssh_client_pubkey_accepted_key_types` (list): Public key types the client accepts for auth (same default set as above).
- `ssh_client_password_authentication` (str): Client password auth fallback (default: `"no"`).
- `ssh_client_challenge_response_auth` (str): Client challenge-response auth (default: `"no"`).
- `ssh_client_pubkey_authentication` (str): Client public key auth (default: `"yes"`).
- `ssh_client_strict_host_key_checking` (str): Host key verification policy — `yes`, `no`, `ask` (default: `ask`).
- `ssh_client_verify_host_key_dns` (str): Verify host keys via DNS SSHFP records (default: `"yes"`).
- `ssh_client_forward_agent` (str): Forward the local ssh-agent (default: `"no"`).
- `ssh_client_forward_x11` (str): Forward X11 (default: `"no"`).
- `ssh_client_forward_x11_trusted` (str): Trust forwarded X11 clients (default: `"no"`).
- `ssh_client_server_alive_interval` (int): Seconds between client keepalives to the server (default: `300`).
- `ssh_client_server_alive_count_max` (int): Unanswered keepalives before the client disconnects (default: `2`).
- `ssh_client_tcp_keep_alive` (str): Send TCP keepalives (default: `"yes"`).
- `ssh_client_compression` (str): Request compression (default: `"no"`).
- `ssh_client_log_level` (str): Client logging verbosity (default: `INFO`).
- `ssh_client_hash_known_hosts` (str): Hash entries added to `known_hosts` (default: `"yes"`).

Example Playbook
-----------------
Basic hardening on default settings:

```yaml
- hosts: all
  become: true
  roles:
    - role: iamenr0s.ansible_role_ssh_hardening
```

Custom port and access control:

```yaml
- hosts: production_servers
  become: true
  vars:
    ssh_port: 2222
    ssh_max_auth_tries: 2
    ssh_max_sessions: 1
    ssh_allow_groups:
      - ssh-users
      - administrators
    ssh_deny_users:
      - guest
    ssh_banner_content: |
      ================================================================================
                                  PRODUCTION SYSTEM
      ================================================================================
      This is a production system. Unauthorized access is strictly prohibited.
      ================================================================================
  roles:
    - role: iamenr0s.ansible_role_ssh_hardening
```

High-security configuration with per-user `Match` overrides:

```yaml
- hosts: secure_servers
  become: true
  vars:
    ssh_port: 2222
    ssh_permit_root_login: "no"
    ssh_max_auth_tries: 1
    ssh_max_sessions: 1
    ssh_allow_users:
      - admin
      - security-officer
    ssh_log_level: VERBOSE
    ssh_match_blocks:
      - condition: "User backup"
        settings:
          ForceCommand: "/usr/local/bin/backup-only.sh"
          AllowTcpForwarding: "no"
  roles:
    - role: iamenr0s.ansible_role_ssh_hardening
```

Notes
-----
- The role backs up the existing `sshd_config` (with a timestamp suffix) before deploying the hardened one, and validates the new config with `sshd -t` via the template's `validate` option before it's ever written live.
- Changing `ssh_port`, `ssh_permit_root_login`, or `ssh_password_authentication` can lock you out if your current session relies on the old settings — ensure console/out-of-band access before applying disruptive changes to a reachable host.
- DSA host keys are removed unconditionally as part of hardening; if a client only supports DSA it will no longer be able to connect.

Contributing & Security
-----------------------
- Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
- Report vulnerabilities privately per [SECURITY.md](SECURITY.md); do not open public issues for them.

CI & Release (maintainers)
---------------------------
A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

The Galaxy API key lives in the `galaxy` GitHub environment, which only `v*` tags may target. One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --env galaxy --repo iamenr0s/ansible-role-ssh-hardening

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --env galaxy --repo iamenr0s/ansible-role-ssh-hardening
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish.

License
-------
MIT

Author Information
-------------------
Author: iamenr0s
Galaxy: `iamenr0s.ansible_role_ssh_hardening`

Changelog
---------
See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.
