# Grafana Provisioning

## Overview

Grafana resources are provisioned from files managed in the infrastructure
Git repository.

Manual recreation of the Loki data source and logging dashboard is not
required after rebuilding Grafana.

## Source templates

```text
roles/monitoring_stack/templates/grafana.ini.j2
roles/monitoring_stack/templates/datasource-loki.yml.j2
roles/monitoring_stack/templates/dashboard-provider.yml.j2
roles/monitoring_stack/templates/centralised-logs.json.j2
```

## Managed files

```text
/opt/infrastructure/monitoring/grafana/grafana.ini
/opt/infrastructure/monitoring/grafana/provisioning/datasources/loki.yml
/opt/infrastructure/monitoring/grafana/provisioning/dashboards/dashboards.yml
/opt/infrastructure/monitoring/grafana/dashboards/centralised-logs.json
```

## Provisioning flow

```text
Git
  ↓
Ansible templates
  ↓
Monitoring runtime directory
  ↓
Docker bind mounts
  ↓
Grafana provisioning system
```

## Loki data source

The data source uses the stable UID:

```text
loki
```

Dashboard panels refer to that UID rather than a generated internal ID.

## Dashboard variables

### Environment

```text
label_values(environment)
```

### Host

```text
label_values({environment=~"$environment"}, host)
```

### Service

```text
label_values(
  {environment=~"$environment", host=~"$host"},
  service
)
```

These variables will populate once Alloy begins forwarding labelled logs.

## Editing dashboards

Provisioned dashboards should not be maintained solely through the Grafana
interface.

To make a permanent change:

1. edit the JSON template in Git;
2. validate the JSON;
3. run the monitoring playbook;
4. verify the dashboard;
5. commit the change.

## Deployment command

```bash
ansible-playbook \
  -i inventories/monitoring/hosts.ini \
  --vault-id monitoring@~/.config/ansible-vault/monitoring.pass \
  playbooks/monitoring.yml
```
