# Ansible Role: SSH Hardening

A comprehensive Ansible role for hardening SSH daemon configuration with security best practices. This role implements industry-standard security measures to protect SSH services against common attacks and vulnerabilities.

## Features

- **Strong Cryptography**: Uses only secure ciphers, MACs, and key exchange algorithms
- **Authentication Hardening**: Configures secure authentication methods and restrictions
- **Access Control**: Implements user/group-based access restrictions
- **Logging & Monitoring**: Enhanced logging for security monitoring
- **Protocol Security**: Disables insecure protocols and features
- **Client Configuration**: Hardens SSH client settings
- **Compliance Ready**: Aligned with CIS benchmarks and NIST guidelines

## Requirements

- Ansible 2.9 or higher
- Target systems with OpenSSH installed
- Root or sudo access on target systems

## Supported Platforms

- Ubuntu 18.04, 20.04, 22.04
- Debian 10, 11, 12
- RHEL 7, 8, 9
- Rocky Linux 8, 9
- AlmaLinux 8, 9
- Fedora 35+

## Role Variables

### Basic Configuration

```yaml
# Network settings
ssh_port: 22
ssh_listen_addresses: []  # Listen on all interfaces by default

# Authentication
ssh_permit_root_login: "no"
ssh_password_authentication: "no"
ssh_pubkey_authentication: "yes"
ssh_max_auth_tries: 3
```

### Security Settings

```yaml
# Strong cryptography (defaults to secure algorithms only)
ssh_ciphers:
  - chacha20-poly1305@openssh.com
  - aes256-gcm@openssh.com
  - aes128-gcm@openssh.com

ssh_macs:
  - hmac-sha2-256-etm@openssh.com
  - hmac-sha2-512-etm@openssh.com

# Access control
ssh_allow_users: []      # Specific users allowed
ssh_deny_users: []       # Specific users denied
ssh_allow_groups: []     # Specific groups allowed
ssh_deny_groups: []      # Specific groups denied
```

### Advanced Configuration

```yaml
# Banner configuration
ssh_banner_enabled: true
ssh_banner_content: "Custom banner message"

# Client alive settings
ssh_client_alive_interval: 300
ssh_client_alive_count_max: 2

# Forwarding restrictions
ssh_allow_agent_forwarding: "no"
ssh_allow_tcp_forwarding: "no"
ssh_x11_forwarding: "no"
```

## Dependencies

None. This role is self-contained.

## Example Playbook

### Basic Usage

```yaml
---
- hosts: servers
  become: yes
  roles:
    - ansible-role-ssh-hardening
```

### Custom Configuration

```yaml
---
- hosts: servers
  become: yes
  vars:
    ssh_port: 2222
    ssh_permit_root_login: "no"
    ssh_password_authentication: "no"
    ssh_allow_users:
      - admin
      - developer
    ssh_banner_enabled: true
    ssh_banner_content: |
      WARNING: Unauthorized access prohibited!
      All activities are monitored and logged.
  roles:
    - ansible-role-ssh-hardening
```

### Advanced Security Configuration

```yaml
---
- hosts: high_security_servers
  become: yes
  vars:
    ssh_port: 2222
    ssh_max_auth_tries: 2
    ssh_max_sessions: 1
    ssh_login_grace_time: 20
    ssh_allow_groups:
      - ssh-users
    ssh_authentication_methods:
      - publickey
    ssh_match_blocks:
      - condition: "User admin"
        settings:
          MaxAuthTries: "5"
          PasswordAuthentication: "yes"
  roles:
    - ansible-role-ssh-hardening
```

## Security Features Implemented

### Cryptographic Hardening
- Disables weak ciphers (3DES, Blowfish, RC4)
- Uses only strong MACs (SHA-2 based)
- Implements secure key exchange algorithms
- Removes weak host key types (DSA)

### Authentication Security
- Disables password authentication by default
- Implements public key authentication
- Configures authentication attempt limits
- Sets secure login grace time

### Access Control
- User and group-based restrictions
- Root login prevention
- Empty password prevention
- Strict mode enforcement

### Protocol Security
- Forces SSH protocol version 2
- Disables X11 forwarding
- Restricts TCP forwarding
- Disables agent forwarding

### Monitoring & Logging
- Enhanced logging configuration
- Security event logging
- Failed authentication tracking

## Security Compliance

This role helps achieve compliance with:

- **CIS Benchmarks**: Implements CIS SSH hardening guidelines
- **NIST Guidelines**: Follows NIST cybersecurity framework recommendations
- **STIG Requirements**: Addresses DoD STIG SSH security requirements
- **PCI DSS**: Supports PCI DSS compliance requirements

## Testing

The role includes validation tasks that:
- Verify SSH configuration syntax
- Test service functionality
- Validate security settings

## Backup and Recovery

The role automatically:
- Creates backups of original configurations
- Validates configurations before applying
- Provides rollback capabilities

## Troubleshooting

### Common Issues

1. **SSH Connection Lost**: Ensure you have alternative access before running
2. **Configuration Validation Failed**: Check custom variables for syntax errors
3. **Service Won't Start**: Review logs and validate configuration

### Recovery Steps

If SSH becomes inaccessible:
1. Use console access or alternative connection method
2. Restore from backup: `cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config`
3. Restart SSH service: `systemctl restart sshd`

## License

MIT

## Author Information

Author: iamenr0s
Galaxy: `iamenr0s.ansible_role_ssh_hardening`

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## Changelog

See `CHANGELOG.md` for version history and release notes.