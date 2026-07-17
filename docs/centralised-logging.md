# Centralised Logging

## Overview

The DevOps home lab uses Grafana Alloy, Loki and Grafana for centralised
log collection and exploration.

## Architecture

```text
web01 Alloy ──┐
              ├── Loki ── Grafana
web02 Alloy ──┘
```

## Components

### Grafana Alloy

Installed natively on:

```text
web01
web02
```

Alloy reads journald and Nginx log files and forwards entries to Loki.

### Loki

Runs as a Docker container on:

```text
automation
```

Address:

```text
192.168.56.10:3100
```

### Grafana

Runs as a Docker container on:

```text
automation
```

Address:

```text
http://192.168.56.10:3000
```

## Collected streams

| Source                     | Service label        |
| -------------------------- | -------------------- |
| Employee Directory journal | `employee-directory` |
| SSH journal                | `ssh`                |
| Nginx access log           | `nginx-access`       |
| Nginx error log            | `nginx-error`        |

## Standard labels

Every collected stream includes:

```text
environment
host
service
```

Examples:

```text
environment=production
host=web01
service=nginx-access
```

```text
environment=staging
host=web02
service=employee-directory
```

## Alloy configuration

```text
/etc/alloy/config.alloy
```

## Alloy service configuration

```text
/etc/default/alloy
```

## Alloy persistent storage

```text
/var/lib/alloy
```

This directory stores component data such as file-read positions.

## Network security

Alloy sends logs outbound to Loki.

No inbound Alloy firewall port is open.

The Alloy debugging interface binds only to:

```text
127.0.0.1:12345
```

Loki permits ingestion only from approved Host-Only addresses through UFW.

## Deployment

```bash
ansible-playbook \
  -i inventories/observability/hosts.ini \
  playbooks/log-agents.yml
```

## Verification

```bash
ansible-playbook \
  -i inventories/observability/hosts.ini \
  playbooks/verify-logging.yml
```

## Useful LogQL

### Production

```logql
{environment="production"}
```

### Staging

```logql
{environment="staging"}
```

### Employee Directory

```logql
{service="employee-directory"}
```

### Nginx errors

```logql
{service="nginx-error"}
```

### Error search

```logql
{environment=~"production|staging"}
  |~ "(?i)error|failed|exception|critical"
```

## Troubleshooting order

1. Verify the source service or log file.
2. Verify Alloy is active.
3. Validate `/etc/alloy/config.alloy`.
4. Confirm Alloy permissions.
5. Confirm Loki is reachable.
6. Inspect Alloy’s journal.
7. Query Loki directly.
8. Inspect Grafana provisioning.
