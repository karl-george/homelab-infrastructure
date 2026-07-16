# Secrets Management

## Overview

Employee Directory secrets are encrypted using Ansible Vault.

Encrypted variables are stored in Git, while vault password files remain only on the Automation VM.

## Encrypted Files

Production:

```text
inventories/production/group_vars/webservers/vault.yml
```

Staging:

```text
inventories/staging/group_vars/webservers/vault.yml
```

## Vault Passowrd Files

Production:

```text
~/.config/ansible-vault/production.pass
```

Staging:

```text
~/.config/ansible-vault/staging.pass
```

These files:

- are not stored in the repository
- use mode `0600`
- are backed up in a trusted password manager
- use different values for each environment

## Deployment Commands

### Staging

```bash
ansible-playbook -i inventories/staging/hosts.ini --vault-id staging@~/.config/ansible-vault/staging.pass playbooks/webservers.yml
```

### Production

```bash
ansible-playbook -i inventories/production/hosts.ini --vault-id production@~/.config/ansible-vault/production.pass playbooks/webservers.yml
```

## Variable Convention

Encrypted variables use the `vault_` prefix:

```yaml
vault_employee_app_secret_key
```

Roles consume a non-vault facing variable:

```yaml
employee_app_secret_key
```

The environment inventory maps one to the other

## Managed Environment File

Ansible deploys:

```text
/etc/employee-directory/employee-directory.env
```

Ownership and Permissions:

```text
root:root 0600
```

The systemd service references it through:

```ini
EnvironmentFile=/etc/employee-directory/employee-directory.env
```

## Output Protection

Secret-bearing Ansible tasks use:

```yaml
no-log: true
```

Template diffs are disabled for the secret environment file.

Avoid displaying:

- full resolved inventory variables
- decrypted vault contents
- systemd process environments
- secret environment-file contents

## Secret Rotation

1. Edit the appropriate vault file
2. Replace the secret
3. Run the environment playbook
4. Verify health
5. Commit the updated encrypted vault file
6. Record the rotation date without recording the value

## Lost Vault Password

If a vault password is lost and no protected backup exists, the encrypted file cannot be recovered.

Create a new vault file and rotate the deployed secret.
