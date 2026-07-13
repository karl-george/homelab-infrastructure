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
