# Centralised Logging Operations Runbook

## Purpose

This runbook describes routine operation and troubleshooting of the
Grafana Alloy, Loki and Grafana logging platform.

## Service locations

| Component | Host       | Runtime |
| --------- | ---------- | ------- |
| Grafana   | automation | Docker  |
| Loki      | automation | Docker  |
| Alloy     | web01      | systemd |
| Alloy     | web02      | systemd |

## Grafana

```text
http://192.168.56.10:3000
```

## Routine verification

### Monitoring containers

```bash
sudo docker compose \
  -f /opt/infrastructure/monitoring/compose.yml \
  ps
```

### Loki readiness

```bash
curl -i http://192.168.56.10:3100/ready
```

### Alloy agents

```bash
ansible log_agents \
  -i inventories/observability/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl is-active alloy"
```

### End-to-end verification

```bash
ansible-playbook \
  -i inventories/observability/hosts.ini \
  playbooks/verify-logging.yml
```

## Useful LogQL queries

### All production logs

```logql
{environment="production"}
```

### All staging logs

```logql
{environment="staging"}
```

### Application logs

```logql
{service="employee-directory"}
```

### Nginx errors

```logql
{service="nginx-error"}
```

### HTTP 4xx and 5xx entries

```logql
{service="nginx-access"}
  | pattern `<remote_addr> - <remote_user> [<time_local>] "<method> <request_uri> <protocol>" <status> <body_bytes_sent> "<http_referer>" "<http_user_agent>"`
  | status =~ `4..|5..`
```

### Log count by service

```logql
sum by (service) (
  count_over_time(
    {environment=~"production|staging"}[5m]
  )
)
```

## Restart procedure

### Restart Grafana and Loki

```bash
sudo docker compose \
  -f /opt/infrastructure/monitoring/compose.yml \
  restart
```

Prefer rerunning the Ansible monitoring playbook when configuration changed.

### Restart Alloy

```bash
ansible log_agents \
  -i inventories/observability/hosts.ini \
  -b \
  -m ansible.builtin.systemd \
  -a "name=alloy state=restarted"
```

## Logs

### Grafana container

```bash
sudo docker logs grafana --tail 100
```

### Loki container

```bash
sudo docker logs loki --tail 100
```

### Alloy

```bash
ansible log_agents \
  -i inventories/observability/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "journalctl -u alloy -n 100 --no-pager"
```

## Storage

Persistent volumes:

```text
monitoring_grafana_data
monitoring_loki_data
```

Inspect:

```bash
sudo docker volume inspect monitoring_grafana_data
sudo docker volume inspect monitoring_loki_data
```

Deleting either volume deletes its stored data.

Do not run:

```bash
docker compose down --volumes
```

unless deliberately destroying monitoring data.

## Retention

Loki retains logs for:

```text
168h
```

which equals seven days.

The retention setting is managed through:

```text
loki_retention_period
```

## Configuration changes

All permanent changes must be made in:

```text
~/projects/homelab-infrastructure
```

Then applied with Ansible.

Do not rely on dashboard changes made only through the Grafana user
interface.
