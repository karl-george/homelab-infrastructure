# Nginx Role

## Purpose

The `nginx` role installs and configures Nginx as the reverse proxy for the Employee Directory application.

## Managed State

The role:

- Installs Nginx;
- Ensures the service is enabled and running;
- Generates the Employee Directory site configuration;
- Enables the site with a symbolic link;
- Disables the default site;
- Validates the complete Nginx configuration;
- Reloads Nginx only when configuration changes;

## Variables

### `nginx_site_name`

Name of the Nginx site configuration.

Default:

```yaml
nginx_site_name: employee-directory
```

### `nginx_listen_port`

Public HTTP port.

Default:

```yaml
nginx_listen_port: 80
```

### `nginx_back_end_address`

Address used to contact Gunicorn

Default:

```yaml
nginx_backend_address: 127.0.0.1
```

### `nginx_backend_port`

Port used to contact Gunicorn

Default:

```yaml
nginx_backend_port: 8000
```

### `nginx_security_headers`

List of HTTP response headers added by Nginx.

## Handler

The role includes a handler that reloads Nginx the managed configuration changes.

A reload preserves exisiting connections while applying the new configuration.

## Validation

```bash
ansible-playbook playbooks/webservers.yml --syntax-check
ansible-playbook playbooks/webservers.yml --check --diff
ansible-playbook playbooks/webservers.yml
curl -I http://192.168.56.40
```
