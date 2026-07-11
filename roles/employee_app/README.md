# Employee Application Role

## Purpose

The `employee_app` role deploys and manages the Employee Directory Flask application.

## Managed State

The role:

- installs required operating-system packages;
- creates the application directory;
- clones or updates the Git repository;
- creates a Python virtual environment;
- installs Python dependencies;
- deploys the systemd service unit;
- reloads systemd when the unit changes;
- restarts Gunicorn when application-related resources change;
- enables and starts the service;
- waits for the backend port;
- verifies the HTTP response.

## Application Location

```text
/opt/apps/employee-directory
```

## Source Directory

```text
/opt/apps/employee-directory/app
```

## Virtual Environment

```text
/opt/apps/employee-directory/app/venv
```

## Service

```text
employee-directory.service
```

## Backend Listener

```text
127.0.0.1:8000
```

## Important Variables

### `employee_app_repo`

Git repository URL.

### `employee_app_version`

Git branch, tag or commit to deploy.

### `employee_app_workers`

Number of Gunicorn workers.

### `employee_app_bind_address`

Gunicorn bind address.

### `employee_app_bind_port`

Gunicorn bind port.

## Deployment

```bash
ansible-playbook playbooks/webservers.yml
```

## Verification

```bash
curl -I http://192.168.56.40
ansible web01 -b -m command -a "systemctl status employee-directory --no-pager"
```
