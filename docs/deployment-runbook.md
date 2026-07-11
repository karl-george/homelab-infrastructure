## Automated Deployment

The Employee Directory application is deployed through Ansible from the automation VM.

### Deployment command

```bash
cd ~/projects/homelab-infrastructure
ansible-playbook playbooks/webservers.yml
```

### Automated actions

The playbook:

1. applies the common server baseline;
2. clones or updates the application repository;
3. creates the virtual environment;
4. installs Python dependencies;
5. deploys the systemd service;
6. restarts Gunicorn when required;
7. configures Nginx;
8. validates the backend and public HTTP endpoints.

### Post-deployment verification

```bash
curl -I http://192.168.56.40
ansible web01 -b -m command -a "systemctl status employee-directory --no-pager"
ansible web01 -b -m command -a "nginx -t"
```

### Rollback

Set `employee_app_version` to the previous known-good Git commit or tag and rerun the playbook.

Example:

```yaml
employee_app_version: "PREVIOUS_COMMIT_HASH"
```

Then:

```bash
ansible-playbook playbooks/webservers.yml
```
