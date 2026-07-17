# Employee Directory Troubleshooting

## Ansible cannot reach the host

Test SSH:

```bash
ssh base@192.168.56.40
```

Test Ansible:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.ping
```

Check:

- VM power state;
- Host-Only adapter;
- IP address;
- UFW OpenSSH rule;
- SSH host-key trust;
- SSH key authentication.

---

## Become fails

Test:

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

## Git deployment fails

Inspect:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.command \
  -a "git -C /opt/apps/employee-directory status"
```

Common causes:

- requested branch or tag does not exist;
- repository contains local edits;
- repository URL is wrong;
- GitHub cannot be reached;
- authentication is required for a private repository.

---

## Gunicorn executable is missing

Check:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.command \
  -a "/opt/apps/employee-directory/app/venv/bin/gunicorn --version"
```

Confirm Gunicorn exists in:

```text
app/requirements.txt
```

---

## systemd reports `BadUnitSetting`

Validate:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemd-analyze verify /etc/systemd/system/employee-directory.service"
```

Inspect:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl cat employee-directory"
```

Common causes:

- malformed directive;
- invalid path;
- multiline `ExecStart`;
- missing executable;
- invalid quoting.

Correct the Jinja2 template, not the generated server file.

---

## Application service fails

Inspect:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl status employee-directory --no-pager -l"
```

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "journalctl -u employee-directory -n 100 --no-pager"
```

Common causes:

- missing dependency;
- wrong working directory;
- incorrect `app:app` target;
- database permission failure;
- Python exception;
- invalid service path.

---

## `no such table: employees`

Run the application-owned initialisation command:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.command \
  -a "/opt/apps/employee-directory/app/venv/bin/python /opt/apps/employee-directory/app/init_db.py"
```

Verify the database path and ownership.

The deployment role should initialise the schema before starting Gunicorn.

---

## Nginx reports an unknown directive

Run:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "nginx -t"
```

Inspect:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "cat -n /etc/nginx/sites-available/employee-directory"
```

Correct the authoritative template:

```text
roles/nginx/templates/employee-directory.conf.j2
```

---

## Nginx returns `502 Bad Gateway`

Check the backend:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.uri \
  -a "url=http://127.0.0.1:8000/health status_code=200"
```

Check the listener:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "ss -lntp"
```

Check the application journal and Nginx error log.

---

## UFW blocks SSH

Use the VM console through VirtualBox.

Check:

```bash
sudo ufw status numbered
```

Allow SSH:

```bash
sudo ufw allow OpenSSH
```

The Ansible common role must always create the OpenSSH rule before enabling UFW.

---

## Playbook changes resources on every run

Run:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/webservers.yml \
  --check \
  --diff
```

Identify the task repeatedly reporting `changed`.

Common causes:

- raw shell or command tasks without appropriate `changed_when`;
- generated content that differs on every run;
- unmanaged server edits;
- variable values changing between runs;
- package cache tasks without `cache_valid_time`.

## Staging deploys the wrong branch

Inspect resolved variables:

```bash
ansible-inventory \
  -i inventories/staging/hosts.ini \
  --host web02
```

Expected:

```yaml
employee_app_version: develop
```

Inspect the deployed branch:

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -m ansible.builtin.command \
  -a "git -C /opt/apps/employee-directory branch --show-current"
```

Expected:

```text
develop
```

---

## Production deploys the wrong branch

Inspect:

```bash
ansible-inventory \
  -i inventories/production/hosts.ini \
  --host web01
```

Expected:

```yaml
employee_app_version: main
```

---

## `Failed to checkout develop`

Verify `develop` exists in the application repository:

```bash
cd ~/projects/employee-directory
git ls-remote --exit-code --heads origin develop
```

Do not create the branch in the infrastructure repository.

---

## Health endpoint reports the wrong environment

Inspect the generated service:

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "systemctl cat employee-directory"
```

Expected:

```ini
Environment="DEPLOYMENT_ENVIRONMENT=staging"
```

If the service template changed, confirm systemd was reloaded and the application restarted.

---

## Staging database table is missing

Run:

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -m ansible.builtin.command \
  -a "/opt/apps/employee-directory/app/venv/bin/python /opt/apps/employee-directory/app/init_db.py"
```

Confirm the `develop` branch contains:

```text
app/init_db.py
```

Do not copy production's SQLite file to staging.

---

## Staging change appears in production before promotion

Check whether production was deployed accidentally:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.command \
  -a "git -C /opt/apps/employee-directory rev-parse HEAD"
```

Inspect Git history and deployment logs.

Separate inventories prevent cross-environment targeting, but they cannot protect against an engineer deliberately running the production command.

---

## New untested commits appeared on `develop`

Compare the deployed staging revision:

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -m ansible.builtin.command \
  -a "git -C /opt/apps/employee-directory rev-parse HEAD"
```

with:

```bash
cd ~/projects/employee-directory
git checkout develop
git pull origin develop
git rev-parse HEAD
```

If they differ, redeploy and retest staging before promotion.

---

## One environment modifies the other environment's data

This should not happen because each server has its own SQLite file.

Verify database paths on each host:

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -m ansible.builtin.stat \
  -a "path=/opt/apps/employee-directory/app/employees.db"
```

```bash
ansible web02 \
  -i inventories/staging/hosts.ini \
  -m ansible.builtin.stat \
  -a "path=/opt/apps/employee-directory/app/employees.db"
```

Identical paths on different hosts do not refer to the same physical file.

## Ansible cannot decrypt vault variables

Confirm the correct environment identity was supplied.

Staging:

```bash
--vault-id staging@~/.config/ansible-vault/staging.pass
```

Production:

```bash
--vault-id production@~/.config/ansible-vault/production.pass
```

Check password-file metadata:

```bash
stat -c '%a %U %G %n' ~/.config/ansible-vault/*.pass
```

Expected mode:

```text
600
```

Do not print the password-file contents in shared terminal output.

## Application fails after secret deployment

Check only the environment-file metadata:

```bash
sudo stat -c '%U %G %a %n' \
  /etc/employee-directory/employee-directory.env
```

Confirm the systemd unit contains the correct `EnvironmentFile` path.

Review the application journal, but do not add debugging that prints the
secret.

## Alloy is active but no logs appear

Check its journal:

```bash
sudo journalctl -u alloy -n 100 --no-pager
```

Validate:

```bash
sudo alloy validate /etc/alloy/config.alloy
```

Check Loki:

```bash
curl -i http://192.168.56.10:3100/ready
```

## Alloy cannot read journald

Check:

```bash
id alloy
```

The user should belong to:

```text
adm
systemd-journal
```

Test:

```bash
sudo -u alloy \
  journalctl -u employee-directory.service -n 5 --no-pager
```

## Alloy cannot read Nginx logs

Test:

```bash
sudo -u alloy tail /var/log/nginx/access.log
```

Confirm the `alloy` user belongs to `adm`.

## Loki has no label values

Generate new traffic after Alloy starts:

```bash
curl http://127.0.0.1/health
```

Remember that the file source initially uses:

```text
tail_from_end = true
```

Existing historical file content is not sent during the initial deployment.

## Loki rejects log delivery

Check the Alloy journal for HTTP errors.

Confirm that monitoring-host UFW permits TCP/3100 from:

```text
192.168.56.40
192.168.56.50
```

Confirm Loki listens on:

```text
192.168.56.10:3100
```

## Dashboard shows no data

Set the time range to:

```text
Last 15 minutes
```

Check the Loki data source.

Run a broad Explore query:

```logql
{service=~".+"}
```

Then narrow the query by environment, host or service.

## Alloy interface is unreachable remotely

This is expected.

Alloy binds to:

```text
127.0.0.1:12345
```

Use an SSH tunnel for temporary access:

```bash
ssh \
  -L 12345:127.0.0.1:12345 \
  base@192.168.56.40
```
