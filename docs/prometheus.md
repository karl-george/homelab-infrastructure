# Prometheus Operations

## Overview

Prometheus stores time-series infrastructure metrics for the DevOps home
lab.

It runs as part of the monitoring Docker Compose project on the automation
VM.

## Architecture

```text
web01 Node Exporter ──┐
                      ├──► Prometheus ───► Grafana
web02 Node Exporter ──┘
```

## Listener

```text
127.0.0.1:9090
```

Prometheus does not listen on the Host-Only interface.

## Scrape targets

### Prometheus

```text
127.0.0.1:9090
```

Labels:

```text
environment=monitoring
host=automation
service=prometheus
```

### web01

```text
192.168.56.40:9100
```

Labels:

```text
environment=production
host=web01
service=node-exporter
```

### web02

```text
192.168.56.50:9100
```

Labels:

```text
environment=staging
host=web02
service=node-exporter
```

## Managed configuration

Git template:

```text
roles/monitoring_stack/templates/prometheus.yml.j2
```

Deployed file:

```text
/opt/infrastructure/monitoring/prometheus/prometheus.yml
```

## Container

```text
prometheus
```

Image:

```text
prom/prometheus:v3.13.1
```

## Persistent storage

Docker volume:

```text
monitoring_prometheus_data
```

Container path:

```text
/prometheus
```

## Retention

```text
15 days
or
5 GB
```

The first limit reached controls retention.

## Deployment

```bash
ansible-playbook \
  -i inventories/monitoring/hosts.ini \
  --vault-id monitoring@~/.config/ansible-vault/monitoring.pass \
  playbooks/monitoring.yml
```

## Configuration validation

```bash
cd /opt/infrastructure/monitoring
```

```bash
sudo docker compose run \
  --rm \
  --no-deps \
  --entrypoint promtool \
  prometheus \
  check config \
  /etc/prometheus/prometheus.yml
```

## Readiness

```bash
curl --fail \
  http://127.0.0.1:9090/-/ready
```

## Health

```bash
curl --fail \
  http://127.0.0.1:9090/-/healthy
```

## Active targets

```bash
curl --fail --silent \
  'http://127.0.0.1:9090/api/v1/targets?state=active' \
  | python3 -m json.tool
```

## Useful PromQL

### Target health

```promql
up
```

### Node Exporter health

```promql
up{job="node-exporter"}
```

### Available memory

```promql
node_memory_MemAvailable_bytes{
  job="node-exporter"
}
```

### CPU time

```promql
node_cpu_seconds_total{
  job="node-exporter"
}
```

### Root filesystem capacity

```promql
node_filesystem_avail_bytes{
  job="node-exporter",
  mountpoint="/"
}
```

### Host uptime

```promql
time()
-
node_boot_time_seconds{
  job="node-exporter"
}
```

## Temporary browser access

From the desktop:

```bash
ssh \
  -L 9090:127.0.0.1:9090 \
  base@automation
```

Then open:

```text
http://127.0.0.1:9090
```

Close the SSH connection when finished.

## Logs

```bash
sudo docker logs \
  --tail 100 \
  prometheus
```

Follow logs:

```bash
sudo docker logs \
  --follow \
  prometheus
```

## Restart

```bash
sudo docker restart prometheus
```

Then verify:

```bash
curl --fail \
  http://127.0.0.1:9090/-/ready
```

## Troubleshooting order

1. Confirm the Prometheus container is running.
2. Confirm Prometheus is ready.
3. Validate `prometheus.yml` with `promtool`.
4. Inspect active targets.
5. Test each Node Exporter endpoint from automation.
6. Inspect UFW on the affected web server.
7. Query `up{job="node-exporter"}`.
8. Inspect Prometheus logs.
9. Verify the Grafana Prometheus data source.
