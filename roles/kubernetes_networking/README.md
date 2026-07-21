# Kubernetes Networking Role

## Purpose

The `kubernetes_networking` role installs the Kubernetes Container Network
Interface (CNI) plugin.

Without a CNI plugin Kubernetes cannot schedule workloads successfully.

This app uses **Flannel** as the cluster networking solution.

---

# Responsibilities

The role is responsible for:

- Downloading the Flannel manifest
- Deploying Flannel
- Waiting for the DaemonSet rollout
- Waiting for the node to become Ready
- Waiting for CoreDNS
- Verifying cluster networking

The role is **not** responsible for:

- Initialising Kubernetes
- Installing Kubernetes packages
- Deploying applications

---

# Role Structure

```
roles/
└── kubernetes_networking/
    ├── defaults/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    └── README.md
```

---

# Variables

| Variable                | Description               |
| ----------------------- | ------------------------- |
| flannel_manifest_url    | Official Flannel manifest |
| flannel_manifest_path   | Local manifest location   |
| flannel_namespace       | Flannel namespace         |
| kubernetes_admin_config | admin.conf location       |
| flannel_rollout_timeout | DaemonSet rollout timeout |
| node_ready_timeout      | Node readiness timeout    |
| coredns_rollout_timeout | CoreDNS rollout timeout   |

---

# Workflow

The role performs the following sequence:

1. Download Flannel manifest
2. Apply manifest
3. Verify namespace creation
4. Wait for DaemonSet rollout
5. Wait for node readiness
6. Wait for CoreDNS rollout
7. Display cluster status

---

# Validation

The role validates that:

- Flannel DaemonSet is Available
- Kubernetes node is Ready
- CoreDNS is Running
- kube-system Pods are healthy

---

# Expected Result

```
kubectl get nodes
```

returns:

```
NAME      STATUS
k8s-lab   Ready
```

---

```
kubectl get pods -n kube-system
```

returns:

```
coredns                     Running
etcd                        Running
kube-apiserver              Running
kube-controller-manager     Running
kube-scheduler              Running
kube-proxy                  Running
```

---

# Dependencies

This role requires:

- kubernetes_node
- kubernetes_control_plane

to have completed successfully.

---

# Networking

Current network configuration:

| Setting      | Value         |
| ------------ | ------------- |
| CNI          | Flannel       |
| Pod CIDR     | 10.244.0.0/16 |
| Service CIDR | 10.96.0.0/16  |
| Runtime      | containerd    |

---

# Future Enhancements

Possible future improvements:

- Replace Flannel with Cilium
- Support Calico
- Dual-stack IPv4/IPv6
- Network policies
- Multi-node overlay networking
