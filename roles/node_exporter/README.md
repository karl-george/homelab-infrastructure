# Node Exporter Role

## Purpose

The `node_exporter` role installs and configures Prometheus Node Exporter
on Linux hosts.

Node Exporter exposes operating-system and hardware metrics for collection
by Prometheus.

## Managed resources

```text
/usr/local/bin/node_exporter
/etc/systemd/system/node_exporter.service
```

The role also manages:

- the `node_exporter` system group;
- the `node_exporter` system user;
- the systemd service;
- the source-restricted UFW rule;
- endpoint and listener validation.

## Version

```text
1.12.0
```

The version is pinned through:

```yaml
node_exporter_version
```

The release archive is validated against the SHA-256 checksums published
with the upstream release.

## Service account

```text
User:  node_exporter
Group: node_exporter
Shell: /usr/sbin/nologin
```

Node Exporter does not run as root.

## Network design

Node Exporter binds to each host's Host-Only address:

```yaml
node_exporter_listen_address: "{{ ansible_host }}"
node_exporter_port: 9100
```

Current endpoints:

```text
web01 → 192.168.56.40:9100
web02 → 192.168.56.50:9100
```

The role rejects wildcard listeners:

```text
0.0.0.0
::
```

## Firewall design

Only the Prometheus host may connect:

```text
192.168.56.10 → TCP/9100
```

No general inbound Node Exporter rule is created.

## Enabled additional collectors

```text
systemd
```

Standard Node Exporter collectors remain enabled.

## Deployment

```bash
ansible-playbook \
  -i inventories/observability/hosts.ini \
  playbooks/metrics-agents.yml
```

## Validation

The role automatically verifies:

- the metrics endpoint returns HTTP 200;
- `node_exporter_build_info` exists;
- the listener uses the configured Host-Only address;
- no IPv4 wildcard listener exists;
- no IPv6 wildcard listener exists;
- the systemd service is active;
- the systemd service is running;
- the systemd service is enabled.

## Manual validation

```bash
curl http://192.168.56.40:9100/metrics
curl http://192.168.56.50:9100/metrics
```

```bash
sudo systemctl status node_exporter
sudo ss -lntp 'sport = :9100'
```
