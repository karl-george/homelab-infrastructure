# Kubernetes Control Plane Role

## Purpose

The `kubernetes_control_plane` role bootstraps a Kubernetes control plane using
`kubeadm`.

Unlike the `kubernetes_node` role, which prepares any machine to become a
Kubernetes node, this role is only executed on control-plane nodes.

The role is designed to be idempotent. Once the cluster has been initialised,
subsequent Ansible runs will not recreate the cluster or regenerate certificates.

---

# Responsibilities

This role is responsible for:

- Validating control-plane configuration
- Deploying the kubeadm configuration
- Configuring the Kubernetes API endpoint
- Pulling Kubernetes control-plane images
- Initialising the cluster using kubeadm
- Waiting for the API server
- Configuring kubectl for the local administrator
- Performing initial cluster validation

It is **not** responsible for:

- Installing Kubernetes packages
- Installing containerd
- Configuring the operating system
- Installing the cluster network plugin
- Deploying workloads

Those responsibilities belong to other roles.

---

# Role Structure

```
roles/
└── kubernetes_control_plane/
    ├── defaults/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    ├── templates/
    │   └── kubeadm-init.yaml.j2
    └── README.md
```

---

# Variables

| Variable                         | Description                    |
| -------------------------------- | ------------------------------ |
| kubernetes_cluster_name          | Kubernetes cluster name        |
| kubernetes_control_plane_address | API server address             |
| kubernetes_control_plane_port    | API server port                |
| kubernetes_pod_network_cidr      | Pod network                    |
| kubernetes_service_network_cidr  | Service network                |
| kubernetes_dns_domain            | Cluster DNS domain             |
| kubernetes_cri_socket            | containerd CRI socket          |
| kubernetes_kubeadm_config_path   | kubeadm configuration          |
| kubernetes_admin_config_path     | admin.conf location            |
| kubernetes_admin_user            | Local Kubernetes administrator |

---

# Templates

## kubeadm-init.yaml.j2

Generates the kubeadm configuration used by:

```
kubeadm init
```

The template contains:

- InitConfiguration
- ClusterConfiguration
- KubeletConfiguration

---

# Idempotence

The role checks for:

```
/etc/kubernetes/admin.conf
```

If the file exists, cluster initialisation is skipped.

This prevents accidental recreation of the control plane.

---

# Validation

The role performs functional validation by:

- Validating the kubeadm configuration
- Waiting for the API server
- Verifying administrator kubeconfig creation
- Querying Kubernetes nodes using kubectl

---

# Expected Result

After successful execution:

```
kubectl get nodes
```

returns:

```
NAME      STATUS
k8s-lab   NotReady
```

This is expected because the CNI plugin has not yet been installed.

---

# Dependencies

This role requires:

- kubernetes_node
- containerd
- kubeadm
- kubelet
- kubectl

to have already been installed.

---

# Future Enhancements

Possible future improvements include:

- High Availability control planes
- External etcd support
- Certificate rotation
- Automatic kubeadm upgrades
- Multi-control-plane clusters
