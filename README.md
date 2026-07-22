# Homelab Infrastructure

A production-style DevOps home lab built to develop practical experience with modern infrastructure automation, configuration management, web application deployment, reverse proxying, containerisation, and centralised monitoring.

This project serves as both a learning platform and a portfolio demonstrating infrastructure managed using Infrastructure as Code (IaC) principles rather than manual configuration.

---

# Project Goals

The primary goals of this repository are to:

- Build infrastructure using repeatable automation.
- Follow Infrastructure as Code best practices.
- Learn industry-standard DevOps tooling.
- Create production-like environments for testing.
- Document every stage of the deployment process.
- Maintain idempotent Ansible playbooks and roles.
- Provide a foundation for future Kubernetes and CI/CD work.

Every change to the infrastructure is version controlled, documented and reproducible.

---

# Lab Architecture

The lab currently consists of multiple Ubuntu Server virtual machines running inside VirtualBox.

```
                    Ubuntu Desktop
                          │
          ┌───────────────┴───────────────┐
          │                               │
     Host Only Network               NAT Network
     (192.168.56.0/24)             Internet Access
          │
 ┌────────┴─────────┐
 │                  │
Automation VM   Other Servers
```

Current infrastructure:

| Host       | Purpose                                    |
| ---------- | ------------------------------------------ |
| automation | Ansible control node and management server |
| web01      | Production web server                      |
| web02      | Staging web server                         |
| monitoring | Grafana, Loki and Alloy logging stack      |

---

# Technologies Used

## Operating System

- Ubuntu Server LTS
- Ubuntu Desktop

## Configuration Management

- Ansible

## Web Stack

- Nginx
- Gunicorn
- Flask

## Monitoring

- Grafana
- Loki
- Alloy

## Containers

- Docker
- Docker Compose

## Version Control

- Git
- GitHub

---

# Repository Structure

```
homelab-infrastructure/
│
├── inventories/
│   ├── production/
│   └── staging/
│
├── playbooks/
│
├── roles/
│   ├── common/
│   ├── docker_engine/
│   ├── employee_app/
│   ├── grafana/
│   ├── loki/
│   ├── alloy/
│   ├── monitoring_stack/
│   ├── nginx/
│   └── ...
│
├── documentation/
│
├── files/
│
├── templates/
│
└── README.md
```

The repository is organised around reusable Ansible roles. Each role is responsible for deploying and configuring a single service or component.

---

# Infrastructure Philosophy

This repository avoids manual server configuration wherever possible.

Infrastructure changes should be made by:

1. Updating Ansible code.
2. Committing the changes.
3. Running the playbooks.
4. Verifying the deployment.

Servers should be considered disposable. A freshly provisioned server should be capable of being configured entirely from this repository.

---

# Current Features

## Common Server Configuration

- Package updates
- Common utilities
- Firewall configuration
- SSH access
- Idempotent provisioning

---

## Employee Directory Application

Deployment of a Flask web application including:

- Git checkout
- Python virtual environment
- Dependency installation
- Gunicorn service
- Systemd integration
- Health checks
- Automatic service restarts

---

## Reverse Proxy

Nginx configured as a reverse proxy providing:

- HTTP request forwarding
- Security headers
- Gunicorn isolation
- Production-style deployment

---

## Docker

Automated installation of:

- Docker Engine
- Docker Compose
- Official Docker repositories
- Docker service management

---

## Centralised Logging

The monitoring stack provides centralised log collection across the lab.

Components:

### Grafana

Provides dashboards and log exploration.

Features include:

- Service log volume
- Log search
- Error filtering
- HTTP request monitoring

---

### Loki

Stores all application and system logs.

Features include:

- Label-based indexing
- Efficient log querying
- LogQL support
- Integration with Grafana

---

### Alloy

Collects logs from monitored servers.

Current sources include:

- Nginx access logs
- Nginx error logs
- System logs

All logs are labelled with:

- Environment
- Host
- Service

allowing dashboards to filter logs dynamically.

---

# Monitoring Dashboard

The logging dashboard currently provides:

- Log volume by service
- HTTP response status counts
- Recent HTTP errors
- Complete searchable log stream
- Error and failure log filtering

Dashboard variables allow filtering by:

- Environment
- Host
- Service
- Free-text search

---

# Environments

Separate inventories are maintained for:

## Production

Contains the production web server and monitoring infrastructure.

## Staging

Used for validating deployments before production rollout.

This separation mirrors common production deployment practices.

---

# Security

Several basic hardening measures have been implemented:

- UFW firewall
- Reverse proxy architecture
- Gunicorn bound to localhost
- Security response headers
- Principle of least privilege
- SSH-based administration
- Configuration managed through Ansible

---

# Automation

Infrastructure is deployed using idempotent Ansible playbooks.

Running a playbook repeatedly should leave servers in the same desired state without unnecessary changes.

Typical deployment:

```bash
ansible-playbook \
-i inventories/production/hosts.ini \
playbooks/site.yml
```

## Kubernetes Application Deployment

The Employee Directory application can now be deployed to the kubeadm-based
Kubernetes cluster using native Kubernetes manifests.

The deployment is stored under:

```text
kubernetes/employee-directory/
```

It currently includes:

- a dedicated Namespace;
- a Deployment managing the application Pod;
- a ClusterIP Service providing stable internal access;
- Kubernetes-standard application labels;
- rolling update and rollback support;
- application-specific deployment and troubleshooting documentation.
