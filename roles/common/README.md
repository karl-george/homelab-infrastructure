# Common Role

## Purpose

The `common` role establishes the baseline configuration shared by all managed Ubuntu servers in the home lab.

## Managed State

The role:

- Refreshes the APT cache when necessary;
- Installs the standard administration packages;
- Ensures UFW and Chrony are installed;

## Variables

### `common_packages`

List of packages installed on every managed node.

Default value:

```yaml
common_packages:
  - curl
  - git
  - htop
  - tree
  - vim
  - ufw
  - chrony
```

The variable is defined in:

```text
roles/common/defaults/main.yml
```

## Usage

```yaml
---
- name: Configure common server baseline
  hosts: all
  become: true

  roles:
    - common
```

## Validation

```bash
ansible-playbook playbooks/common.yml --syntax-check
ansible-playbook playbooks/common.yml --check --diff
ansible-playbook playbooks/common.yml
```

A second normal run should report no changes when the managed nodes already match the desired state.
