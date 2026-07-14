# Employee Directory Deployment Runbook

## Purpose

This document describes how to deploy, validate and roll back the Employee Directory application.

The production application is deployed to `web01` through Ansible from the `automation` VM.

Manual application deployment is no longer the approved workflow.

---

## Architecture

```text
Ubuntu desktop
      |
      | SSH
      v
automation VM
      |
      | Ansible over SSH
      v
web01
      |
      ├── UFW
      ├── Nginx :80
      ├── Gunicorn 127.0.0.1:8000
      ├── Flask
      └── SQLite
```

---

## Repositories

### Application repository

```text
~/projects/employee-directory
```

Contains:

- Flask source code;
- templates and static files;
- database initialisation code;
- Python requirements.

### Infrastructure repository

```text
~/projects/homelab-infrastructure
```

Contains:

- inventories;
- playbooks;
- Ansible roles;
- service templates;
- Nginx templates;
- firewall configuration;
- operational documentation.

---

## Prerequisites

Before deployment:

- `web01` must be powered on;
- the Host-Only network must be available;
- SSH key authentication must work;
- Ansible privilege escalation must work;
- application changes must be committed and pushed;
- infrastructure changes must be committed locally or present on the deployment branch;
- required Ansible collections must be installed.

Verify connectivity:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.ping
```

Verify privilege escalation:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "whoami"
```

Expected:

```text
root
```

---

## Pre-deployment checks

### Check application repository

```bash
cd ~/projects/employee-directory

git status
git branch --show-current
git log -1 --oneline
git push origin main
```

### Check infrastructure repository

```bash
cd ~/projects/homelab-infrastructure

git status
git branch --show-current
```

### Validate Ansible syntax

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml \
  --syntax-check
```

### Preview changes

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml \
  --check \
  --diff
```

Check mode is a preview and does not replace a normal deployment and post-deployment validation.

---

## Production deployment

From the infrastructure repository:

```bash
cd ~/projects/homelab-infrastructure
```

Run:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

A successful run must show:

```text
failed=0
unreachable=0
```

---

## Automated deployment actions

The playbook:

1. installs common administration packages;
2. configures UFW safely;
3. allows SSH before enabling UFW;
4. allows HTTP on the web server;
5. installs application prerequisites;
6. clones or updates the application repository;
7. creates the Python virtual environment;
8. installs Python dependencies;
9. initialises the SQLite database;
10. verifies database ownership;
11. deploys and validates the systemd service;
12. reloads systemd when required;
13. restarts Gunicorn when required;
14. waits for the backend port;
15. calls the backend health endpoint;
16. installs and configures Nginx;
17. validates Nginx;
18. calls the application through Nginx.

---

## Post-deployment validation

### Service state

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl is-active employee-directory"
```

Expected:

```text
active
```

### Backend listener

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "ss -lntp"
```

Expected backend:

```text
127.0.0.1:8000
```

### Backend health

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.uri \
  -a "url=http://127.0.0.1:8000/health status_code=200"
```

### Nginx validation

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "nginx -t"
```

### Public health endpoint

```bash
curl -i http://192.168.56.40/health
```

### Main website

```bash
curl -I http://192.168.56.40/
```

### Firewall

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "ufw status verbose"
```

---

## Application logs

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "journalctl -u employee-directory -n 100 --no-pager"
```

---

## Nginx logs

### Error log

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "tail -50 /var/log/nginx/error.log"
```

### Access log

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "tail -50 /var/log/nginx/access.log"
```

---

## Application rollback

Production currently deploys the revision defined by:

```yaml
employee_app_version
```

To roll back to a known-good commit, temporarily set the variable to that commit hash.

Example:

```yaml
employee_app_version: "3e15a5c2c4a6..."
```

Run:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

Validate the service and public endpoint.

The rollback revision should later be recorded properly in the environment configuration rather than left as an undocumented temporary override.

---

## Configuration rollback

Use Git to revert the faulty infrastructure commit:

```bash
git revert <commit-hash>
```

Push the revert and rerun:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

Do not permanently repair generated files directly on `web01`.

---

## Database recovery

The SQLite database is runtime data and is not reverted through Git.

The temporary transition backup is stored at:

```text
/root/employee-directory-pre-ansible/employees.db
```

A restore must be performed only after confirming that the current database is unusable.

Stop the application:

```bash
sudo systemctl stop employee-directory
```

Back up the failed database:

```bash
sudo cp \
  /opt/apps/employee-directory/app/employees.db \
  /opt/apps/employee-directory/app/employees.db.failed
```

Restore:

```bash
sudo cp \
  /root/employee-directory-pre-ansible/employees.db \
  /opt/apps/employee-directory/app/employees.db
```

Correct ownership:

```bash
sudo chown \
  base:base \
  /opt/apps/employee-directory/app/employees.db
```

Start the application:

```bash
sudo systemctl start employee-directory
```

Verify:

```bash
curl -i http://127.0.0.1:8000/health
```

A formal scheduled backup process will replace this temporary transition procedure in a later ticket.

---

## Definition of a successful deployment

A deployment is successful only when:

- the playbook completes without failure;
- Gunicorn is active;
- Gunicorn listens only on localhost;
- Nginx configuration validates;
- UFW is active;
- SSH and HTTP rules are present;
- backend health returns HTTP 200;
- public health returns HTTP 200;
- the application operates correctly in a browser;
- logs contain no unexplained errors.

## Environment-specific deployments

### Production

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

Production targets:

```text
web01
```

Production deploys:

```text
main
```

### Staging

```bash
ansible-playbook \
  -i inventories/staging/hosts.ini \
  playbooks/webservers.yml
```

Staging targets:

```text
web02
```

Staging deploys:

```text
develop
```

### Safety rule

Always specify the inventory explicitly.

Do not depend on an implicit default inventory when multiple environments exist.

Before deployment, inspect the target:

```bash
ansible-inventory \
  -i inventories/staging/hosts.ini \
  --graph
```

or:

```bash
ansible-inventory \
  -i inventories/production/hosts.ini \
  --graph
```
