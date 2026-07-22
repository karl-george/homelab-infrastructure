# Homelab Infrastructure

A production-style DevOps homelab built to develop practical experience with infrastructure automation, configuration management, containerisation, Kubernetes, observability and application deployment.

The entire environment is managed using **Infrastructure as Code (IaC)** principles, with the objective of creating reproducible, production-inspired infrastructure rather than manually configured virtual machines.

---

# Project Goals

This repository exists to:

- Build infrastructure using repeatable automation.
- Follow Infrastructure as Code best practices.
- Learn industry-standard DevOps tooling.
- Deploy applications using modern infrastructure platforms.
- Create production-like environments for experimentation.
- Document every stage of the deployment process.
- Maintain reusable, idempotent Ansible roles.
- Build a foundation for future cloud and CI/CD projects.

Every infrastructure change is version controlled, documented and reproducible.

---

# Lab Architecture

The lab consists of multiple Ubuntu Server virtual machines running in VirtualBox.

```
                    Ubuntu Desktop
                          │
          ┌───────────────┴───────────────┐
          │                               │
     Host Only Network               NAT Network
     (192.168.56.0/24)             Internet Access
          │
 ┌────────┴──────────────────────────────────────┐
 │                                               │
 │                 Virtual Machines              │
 │                                               │
 │  automation   Ansible Control Node            │
 │  web01        Production Web Server           │
 │  web02        Staging Web Server              │
 │  monitoring   Observability Stack             │
 │  k8s-lab      Kubernetes Cluster              │
 └───────────────────────────────────────────────┘
```

---

# Current Infrastructure

| Host       | Purpose                                                               |
| ---------- | --------------------------------------------------------------------- |
| automation | Ansible control node, Docker build host and Kubernetes administration |
| web01      | Production application server                                         |
| web02      | Staging application server                                            |
| monitoring | Grafana, Loki and Alloy monitoring stack                              |
| k8s-lab    | Single-node Kubernetes cluster                                        |

---

# Technologies Used

## Operating Systems

- Ubuntu Server LTS
- Ubuntu Desktop

## Infrastructure as Code

- Ansible

## Containers

- Docker
- Docker Compose
- GitHub Container Registry (GHCR)

## Container Orchestration

- Kubernetes (kubeadm)
- kubectl

## Web Stack

- Flask
- Gunicorn
- Nginx

## Monitoring & Logging

- Grafana
- Loki
- Alloy

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
│   ├── staging/
│   ├── monitoring/
│   ├── observability/
│   └── kubernetes/
│
├── kubernetes/
│   └── employee-directory/
│
├── playbooks/
│
├── roles/
│
├── documentation/
│
├── templates/
│
├── files/
│
└── README.md
```

Infrastructure is organised into reusable Ansible roles and Kubernetes manifests to encourage modularity and repeatability.

---

# Infrastructure Philosophy

Infrastructure should be reproducible.

Servers are treated as disposable resources and should be rebuilt through automation rather than manually configured.

The normal workflow is:

1. Modify infrastructure code.
2. Commit changes.
3. Execute Ansible playbooks.
4. Validate the deployment.
5. Repeat safely using idempotent automation.

---

# Current Features

## Base Server Configuration

- Package management
- Common utilities
- SSH configuration
- Firewall configuration
- Idempotent provisioning

---

## Employee Directory Application

Automated deployment of a Flask application including:

- Git checkout
- Python virtual environment
- Dependency installation
- Gunicorn service
- Systemd integration
- Health verification
- Automatic service recovery

---

## Reverse Proxy

Nginx configured as a reverse proxy providing:

- Request forwarding
- Security headers
- Gunicorn isolation
- Production deployment model

---

## Docker Platform

Automated installation and configuration of:

- Docker Engine
- Docker Compose
- Docker repositories
- Docker service management

---

## Kubernetes

A single-node Kubernetes cluster built with **kubeadm**.

Current capabilities include:

- Cluster administration using `kubectl`
- Application deployment using Kubernetes manifests
- Namespace isolation
- Deployments
- ReplicaSets
- Pods
- ClusterIP Services
- Rolling updates
- Rollback support
- Container images hosted in GitHub Container Registry

The Employee Directory application is deployed to Kubernetes using manifests stored in:

```
kubernetes/employee-directory/
```

---

## Centralised Logging

The monitoring stack provides centralised log collection.

### Grafana

Provides dashboards for:

- Log exploration
- Error investigation
- Service monitoring

### Loki

Provides:

- Central log storage
- Label-based indexing
- LogQL querying

### Alloy

Collects logs including:

- Nginx access logs
- Nginx error logs
- System logs

Logs are labelled by:

- Environment
- Host
- Service

allowing dashboards to dynamically filter log data.

---

# Monitoring Dashboard

Current dashboards provide:

- Log volume by service
- HTTP response status counts
- Recent HTTP failures
- Searchable log streams
- Error filtering

Dashboard variables allow filtering by:

- Environment
- Host
- Service
- Search query

---

# Deployment Environments

Separate inventories are maintained for:

## Production

Production application deployment.

## Staging

Application validation before production deployment.

This separation mirrors common deployment workflows used in production environments.

---

# Security

Current hardening measures include:

- UFW firewall
- SSH administration
- Reverse proxy architecture
- Gunicorn bound to localhost
- Security response headers
- Non-root application execution where appropriate
- Principle of least privilege
- Infrastructure managed through Ansible

---

# Automation

Infrastructure is deployed using idempotent Ansible playbooks.

Example:

```bash
ansible-playbook \
  -i inventories/production/hosts.ini \
  playbooks/site.yml
```

---

# Kubernetes Deployment

The Employee Directory application is deployed to Kubernetes using native manifests.

Deployment currently includes:

- Namespace
- Deployment
- ReplicaSet
- Pod
- ClusterIP Service

Application images are:

- Containerised with Docker
- Published to GitHub Container Registry
- Pulled directly by Kubernetes

Deployments support:

- Rolling updates
- Rollbacks
- Image versioning
