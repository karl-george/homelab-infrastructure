# Grafana Alloy Role

## Purpose

The `alloy` role installs and configures Grafana Alloy as the log collector
for Employee Directory web servers.

## Managed hosts

```text
web01 → production
web02 → staging
```

## Collected logs

### Employee Directory journal

```text
_SYSTEMD_UNIT=employee-directory.service
```

Loki service label:

```text
employee-directory
```

### SSH journal

```text
_SYSTEMD_UNIT=ssh.service
```

Loki service label:

```text
ssh
```

### Nginx access log

```text
/var/log/nginx/access.log
```

Loki service label:

```text
nginx-access
```

### Nginx error log

```text
/var/log/nginx/error.log
```

Loki service label:

```text
nginx-error
```

## Standard labels

Every forwarded log stream contains:

```text
environment
host
service
```

## Loki destination

```text
http://192.168.56.10:3100/loki/api/v1/push
```

## Service configuration

Alloy configuration:

```text
/etc/alloy/config.alloy
```

Service environment:

```text
/etc/default/alloy
```

Persistent data and read positions:

```text
/var/lib/alloy
```

## Permissions

The `alloy` user belongs to:

```text
adm
systemd-journal
```

These groups permit access to the Nginx logs and system journal.

## Network security

Alloy initiates outbound connections to Loki.

No inbound Alloy firewall rule is created.

The local Alloy HTTP interface listens only on:

```text
127.0.0.1:12345
```

## Deployment

```bash
ansible-playbook \
  -i inventories/observability/hosts.ini \
  playbooks/log-agents.yml
```

## Configuration lifecycle

```text
Generate configuration
        ↓
Validate with alloy validate
        ↓
Restart Alloy
        ↓
Wait for local readiness
```