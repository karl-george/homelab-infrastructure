# Docker Engine Role

## Purpose

The `docker_engine` role installs Docker Engine and the Docker Compose
plugin from Docker's official Ubuntu repository.

## Managed state

The role:

- installs repository prerequisites
- installs Docker's signing key
- configures the Docker APT repository
- installs Docker Engine
- installs containerd
- installs Buildx
- installs the Compose plugin
- enables and starts Docker
- verifies Docker and Compose

## Security

The normal administrative account is not added to the Docker group.

Ansible performs Docker operations with privilege escalation.

The Docker group should be treated as root-equivalent access.

## Validation

```bash
sudo docker version
sudo docker compose version
```
