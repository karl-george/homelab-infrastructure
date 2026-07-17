# Node Exporter Operations

## Overview

Prometheus Node Exporter exposes Linux host metrics from the monitored web
servers.

Current hosts:

```text
web01
web02
```

## Architecture

```text
web01:9100 ──┐
             ├──► Prometheus on automation
web02:9100 ──┘
```

## Endpoints

```text
web01 → http://192.168.56.40:9100/metrics
web02 → http://192.168.56.50:9100/metrics
```

## Access control

Node Exporter binds only to each host's Host-Only interface.

UFW allows connections only from:

```text
192.168.56.10
```

The desktop and other VMs are not approved metrics clients.

## Managed files

```text
/usr/local/bin/node_exporter
/etc/systemd/system/node_exporter.service
```

## Deployment

```bash
ansible-playbook \
  -i inventories/observability/hosts.ini \
  playbooks/metrics-agents.yml
```

## Service status

```bash
sudo systemctl status node_exporter
```

## Service logs

```bash
sudo journalctl \
  -u node_exporter \
  -n 100 \
  --no-pager
```

Follow logs:

```bash
sudo journalctl \
  -u node_exporter \
  -f
```

## Listener validation

On `web01`:

```bash
sudo ss -lntp 'sport = :9100'
```

Expected:

```text
192.168.56.40:9100
```

On `web02`, expected:

```text
192.168.56.50:9100
```

A listener on either of these is incorrect:

```text
0.0.0.0:9100
[::]:9100
```

## Metrics validation

From the automation VM:

```bash
curl --fail \
  http://192.168.56.40:9100/metrics
```

```bash
curl --fail \
  http://192.168.56.50:9100/metrics
```

## Important metrics

Build information:

```text
node_exporter_build_info
```

CPU time:

```text
node_cpu_seconds_total
```

Available memory:

```text
node_memory_MemAvailable_bytes
```

Filesystem capacity:

```text
node_filesystem_size_bytes
node_filesystem_avail_bytes
```

Network traffic:

```text
node_network_receive_bytes_total
node_network_transmit_bytes_total
```

System boot time:

```text
node_boot_time_seconds
```

## Restart

```bash
sudo systemctl restart node_exporter
```

Then validate:

```bash
sudo systemctl is-active node_exporter
```

## Upgrade process

1. review the upstream Node Exporter release;
2. update `node_exporter_version`;
3. run the metrics-agent playbook;
4. verify the installed version;
5. verify the metrics endpoints;
6. verify the restricted listeners;
7. run the playbook a second time;
8. commit the reviewed version change.

Do not manually replace `/usr/local/bin/node_exporter`.

The Ansible role is the source of truth.
