# Employee Application Role

## Purpose

The `employee_app` role deploys and manages the Employee Directory Flask application.

It converts the manual deployment process into a repeatable and idempotent Ansible workflow.

---

## Managed state

The role ensures that:

- required operating-system packages are installed;
- the application root directory exists;
- the application repository is cloned or updated;
- the requested Git revision is deployed;
- a Python virtual environment exists;
- application dependencies are installed;
- the SQLite schema is initialised;
- the database file exists with the correct ownership;
- the systemd service is generated from a template;
- the systemd unit passes validation;
- Gunicorn is enabled and running;
- Gunicorn listens only on localhost;
- the backend health endpoint returns HTTP 200.

---

## Application paths

### Repository

```text
/opt/apps/employee-directory
```

### Application source

```text
/opt/apps/employee-directory/app
```

### Virtual environment

```text
/opt/apps/employee-directory/app/venv
```

### SQLite database

```text
/opt/apps/employee-directory/app/employees.db
```

### systemd unit

```text
/etc/systemd/system/employee-directory.service
```

---

## Important variables

### `employee_app_repo`

Git repository containing the application.

Default:

```yaml
employee_app_repo: "https://github.com/karl-george/employee-directory.git"
```

### `employee_app_version`

Git branch, tag or commit to deploy.

Default:

```yaml
employee_app_version: main
```

Production currently uses `main`.

Later environments may override this value.

### `employee_app_user`

Operating-system user that owns and runs the application.

Default:

```yaml
employee_app_user: base
```

### `employee_app_workers`

Number of Gunicorn worker processes.

Default:

```yaml
employee_app_workers: 3
```

### `employee_app_bind_address`

Gunicorn bind address.

Default:

```yaml
employee_app_bind_address: 127.0.0.1
```

Gunicorn must not bind to `0.0.0.0` because clients should reach the application through Nginx.

### `employee_app_bind_port`

Gunicorn backend port.

Default:

```yaml
employee_app_bind_port: 8000
```

### `employee_app_healthcheck_path`

Application health endpoint.

Default:

```yaml
employee_app_healthcheck_path: /health
```

---

## Deployment order

```text
Install packages
        ↓
Create application directory
        ↓
Clone or update repository
        ↓
Create virtual environment
        ↓
Install Python requirements
        ↓
Initialise SQLite schema
        ↓
Verify database
        ↓
Deploy systemd unit
        ↓
Validate systemd unit
        ↓
Run handlers
        ↓
Enable and start Gunicorn
        ↓
Wait for backend port
        ↓
Verify backend health
```

---

## Handlers

### Reload systemd

Runs when the systemd unit changes.

```yaml
- name: Reload systemd
  ansible.builtin.systemd:
    daemon_reload: true
```

### Restart employee application

Runs when:

- application source changes;
- Python requirements change;
- the systemd unit changes.

The restart occurs only after the generated systemd unit passes validation.

---

## Database initialisation

The role runs:

```bash
/opt/apps/employee-directory/app/venv/bin/python init_db.py
```

from the application source directory.

The initialisation script uses:

```sql
CREATE TABLE IF NOT EXISTS employees
```

It is therefore safe to run repeatedly.

The database is runtime state and must not be committed to Git.

---

## Usage

```yaml
---
- name: Configure Employee Directory web servers
  hosts: webservers
  become: true

  roles:
    - common
    - web_firewall
    - employee_app
    - nginx
```

---

## Deployment command

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

---

## Validation

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl is-active employee-directory"
```

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.uri \
  -a "url=http://127.0.0.1:8000/health status_code=200"
```

```bash
curl -I http://192.168.56.40/
```

---

## Ownership rule

The following resources must not be edited manually during normal operation:

```text
/opt/apps/employee-directory
/opt/apps/employee-directory/app/venv
/etc/systemd/system/employee-directory.service
```

Application changes belong in the application Git repository.

Deployment and service changes belong in the infrastructure Git repository.
