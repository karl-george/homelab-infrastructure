# UFW Firewall Management

## Overview

UFW is enabled on all managed VMs.

The common Ansible role configures the baseline firewall policy:

- deny incoming traffic by default;
- allow outgoing traffic by default;
- allow OpenSSH;
- enable UFW.

The OpenSSH rule is created before the firewall is enabled to prevent the automation VM from losing SSH access.

## Web servers

The `web_firewall` role allows inbound HTTP traffic on TCP port 80.

HTTPS on TCP port 443 remains disabled until TLS is introduced.

## Source of truth

Firewall rules must be changed through Ansible.

Rules should not be added manually on managed hosts unless performing emergency recovery.

## Validation

```bash
ansible all \
  -b \
  -m ansible.builtin.command \
  -a "ufw status verbose"
```
