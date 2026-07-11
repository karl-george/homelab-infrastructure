# Nginx Configuration Automation

## Overview

Nginx configuration for the Employee Directory is managed through the `nginx` Ansible role.

The authoritative source file is:

```text
roles/nginx/templates/employee-directory.conf.j2
```

The generated server file is:

```text
/etc/nginx/sites-available/employee-directory
```

## Deployment Flow

```text
Git template
    |
    v
Ansible template task
    |
    v
Nginx configuration
    |
    v
nginx -t
    |
    v
Reload handler
```

## Operational Rule

The generated file on the server must not be edited manually.

All configuration changes should be:

1. made in Git
2. reviewed with check and diff mode
3. deployed through Ansible
4. validated after deployment

## Rollback

To restore a previous configuration:

1. revert the relevant git commit
2. rerun the Ansible playbook
3. verify `nginx -t`
4. test the application
