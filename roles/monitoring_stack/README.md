# Monitoring Stack Role

## Purpose

The `monitoring_stack` role deploys Grafana and Loki on the automation VM
using Docker Compose.

Grafana configuration, data sources and dashboards are provisioned from
Git-managed files.

## Managed services

### Grafana

```text
192.168.56.10:3000
```

Provides the authenticated log-search interface.

### Loki

```text
192.168.56.10:3100
```

Stores and queries logs received from monitored hosts.

## Runtime directory

```text
/opt/infrastructure/monitoring
```

## Persistent volumes

```text
monitoring_grafana_data
monitoring_loki_data
```

## Provisioned data source

```text
Name: Loki
UID: loki
Default: true
```

Grafana reaches Loki through:

```text
http://127.0.0.1:3100
```

because both containers use host networking.

## Provisioned dashboard

```text
Folder: Centralised Logging
Dashboard: Centralised Logs
UID: centralised-logs
```

The dashboard includes filters for:

- environment;
- host;
- service;
- free-text search.

## Source-of-truth rule

Provisioned Grafana resources must be changed in Git and deployed through
Ansible.

Changes made only through the Grafana interface are not authoritative and
may be overwritten by provisioning.

## Security

- Anonymous Grafana access is disabled.
- User self-registration is disabled.
- The administrator password is stored in Ansible Vault.
- Grafana is accessible only from the desktop Host-Only address.
- Loki ingestion is allowed only from approved monitored hosts.
- Loki is not exposed through Docker port publishing.

## Deployment

```bash
ansible-playbook \
  -i inventories/monitoring/hosts.ini \
  --vault-id monitoring@~/.config/ansible-vault/monitoring.pass \
  playbooks/monitoring.yml
```

## Validation

The role automatically verifies:

- Docker Compose configuration;
- Loki readiness;
- Grafana health;
- provisioned Loki data source;
- provisioned logging dashboard.
