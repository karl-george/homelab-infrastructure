# Employee Directory Promotion Runbook

## Purpose

This document describes how an Employee Directory application change moves from development through staging and into production.

The approved workflow is:

```text
Feature branch
      ↓
develop
      ↓
Staging deployment
      ↓
Staging validation
      ↓
main
      ↓
Production deployment
      ↓
Production validation
```

---

## Branch responsibilities

### `develop`

Represents the application revision intended for staging validation.

Staging deploys:

```yaml
employee_app_version: develop
```

### `main`

Represents the production-ready application revision.

Production deploys:

```yaml
employee_app_version: main
```

### Feature branches

Application work should be completed on temporary feature branches created from `develop`.

Example:

```bash
git checkout develop
git pull origin develop
git checkout -b feature/example-change
```

---

## Step 1 — Implement the change

Work in the application repository:

```bash
cd ~/projects/employee-directory
```

Create the feature branch from `develop`:

```bash
git checkout develop
git pull origin develop
git checkout -b feature/example-change
```

Implement and test the application change locally.

Review:

```bash
git status
git diff
```

Commit:

```bash
git add .
git commit -m "feat: describe the application change"
```

Push:

```bash
git push -u origin feature/example-change
```

---

## Step 2 — Merge the feature into `develop`

```bash
git checkout develop
git pull origin develop
git merge --no-ff feature/example-change
git push origin develop
```

Delete the completed feature branch:

```bash
git branch -d feature/example-change
git push origin --delete feature/example-change
```

---

## Step 3 — Record the staging candidate

```bash
git rev-parse develop
```

Save:

```text
Staging candidate revision: <commit hash>
```

Confirm GitHub has the revision:

```bash
git ls-remote --heads origin develop
```

---

## Step 4 — Preview the staging deployment

Move to the infrastructure repository:

```bash
cd ~/projects/homelab-infrastructure
```

Check inventory variables:

```bash
ansible-inventory \
  -i inventories/staging/hosts.ini \
  --host web02
```

Run syntax validation:

```bash
ansible-playbook \
  -i inventories/staging/hosts.ini \
  playbooks/webservers.yml \
  --syntax-check
```

Preview:

```bash
ansible-playbook \
  -i inventories/staging/hosts.ini \
  playbooks/webservers.yml \
  --check \
  --diff
```

---

## Step 5 — Deploy staging

```bash
ansible-playbook \
  -i inventories/staging/hosts.ini \
  playbooks/webservers.yml
```

Required recap:

```text
failed=0
unreachable=0
```

---

## Step 6 — Validate staging

### Environment identity

```bash
curl -s http://192.168.56.50/health | python3 -m json.tool
```

Expected:

```json
{
  "environment": "staging",
  "status": "healthy"
}
```

### Application service

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl is-active employee-directory"
```

### Nginx

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "nginx -t"
```

### Firewall

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "ufw status verbose"
```

### Application functionality

Verify in the browser:

```text
http://192.168.56.50
```

Test:

- page rendering;
- CSS;
- add operation;
- delete operation;
- persistent staging data;
- application logs;
- Nginx logs.

---

## Step 7 — Record the approved staging revision

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -m ansible.builtin.command \
  -a "git -C /opt/apps/employee-directory rev-parse HEAD"
```

Save:

```text
Approved staging revision: <commit hash>
```

Before promotion, verify local `develop` still points to the same commit:

```bash
cd ~/projects/employee-directory
git checkout develop
git pull origin develop
git rev-parse HEAD
```

Stop if the hash differs.

Any new commit must be deployed and tested in staging before promotion.

---

## Step 8 — Promote `develop` into `main`

Update `main`:

```bash
git checkout main
git pull origin main
```

Merge:

```bash
git merge --no-ff develop
```

Push:

```bash
git push origin main
```

Record:

```bash
git rev-parse HEAD
```

Save:

```text
Production candidate revision: <commit hash>
```

---

## Step 9 — Preview production

Move to the infrastructure repository:

```bash
cd ~/projects/homelab-infrastructure
```

Confirm production variables:

```bash
ansible-inventory \
  -i inventories/production/hosts.ini \
  --host web01
```

Run:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml \
  --syntax-check
```

Preview:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml \
  --check \
  --diff
```

---

## Step 10 — Deploy production

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

Required recap:

```text
failed=0
unreachable=0
```

---

## Step 11 — Validate production

### Environment identity

```bash
curl -s http://192.168.56.40/health | python3 -m json.tool
```

Expected:

```json
{
  "environment": "production",
  "status": "healthy"
}
```

### Application service

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl is-active employee-directory"
```

### Nginx

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "nginx -t"
```

### Browser test

```text
http://192.168.56.40
```

Verify:

- promoted feature is present;
- production data remains;
- add and delete operations work;
- no unexpected application or Nginx errors exist.

---

## Promotion approval requirements

Do not promote a change unless:

- the exact staging revision was recorded;
- the exact staging revision passed validation;
- no untested commits entered `develop`;
- production inventory still targets only `web01`;
- staging inventory still targets only `web02`;
- application changes are committed and pushed;
- infrastructure syntax checks pass;
- staging is healthy and idempotent.

---

## Emergency production rollback

### Git revert approach

Identify the production promotion merge:

```bash
cd ~/projects/employee-directory
git checkout main
git pull origin main
git log --oneline --graph --decorate -15
```

Revert:

```bash
git revert -m 1 <promotion-merge-commit>
git push origin main
```

Redeploy:

```bash
cd ~/projects/homelab-infrastructure

ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml
```

### Temporary revision pin

Edit:

```text
inventories/production/group_vars/webservers.yml
```

Temporarily replace:

```yaml
employee_app_version: main
```

with:

```yaml
employee_app_version: "<known-good-commit>"
```

Deploy production.

The underlying Git history must still be corrected afterwards.

---

## Important database note

Application rollback does not automatically revert:

- data changes;
- deleted records;
- database schema changes.

Database backup and migration planning must be handled separately.
