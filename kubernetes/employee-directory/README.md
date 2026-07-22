# Employee Directory Kubernetes Deployment

## Purpose

This directory contains the native Kubernetes manifests used to deploy the
Employee Directory application.

The deployment demonstrates the core Kubernetes workload resources

- Namespace
- Deployment
- ReplicaSet
- Pod
- ClusterIP Service
- labels and selectors
- rolling updates
- rollbacks

---

## Architecture

```text
Namespace: employee-directory
│
├── Deployment: employee-directory
│   └── ReplicaSet
│       └── Pod
│           └── Employee Directory container
│               └── TCP port 8000
│
└── Service: employee-directory
    ├── ClusterIP
    ├── Service port 80
    └── Pod target port 8000
```

The Service selects the application Pod using these labels:

```yaml
app.kubernetes.io/name: employee-directory
app.kubernetes.io/component: web
```

---

## Manifest files

### `namespace.yaml`

Creates the dedicated `employee-directory` Namespace.

The Namespace keeps the application resources separate from Kubernetes system
components and other workloads.

### `deployment.yaml`

Creates and manages the Employee Directory application Pod.

The Deployment provides:

- declarative workload management;
- automatic Pod replacement;
- ReplicaSet management;
- rolling updates;
- deployment history;
- rollback support.

### `service.yaml`

Creates an internal ClusterIP Service.

The Service provides a stable IP address and DNS name even when the underlying
Pod is recreated with a different IP address.

The application is available inside the cluster at:

```text
http://employee-directory
```

or by its fully qualified service name:

```text
http://employee-directory.employee-directory.svc.cluster.local
```

---

## Apply the manifests

Run the commands from the repository root.

Create the Namespace first:

```bash
kubectl apply \
  -f kubernetes/employee-directory/namespace.yaml
```

Create the Deployment:

```bash
kubectl apply \
  -f kubernetes/employee-directory/deployment.yaml
```

Create the Service:

```bash
kubectl apply \
  -f kubernetes/employee-directory/service.yaml
```

The entire directory can also be applied after the Namespace exists:

```bash
kubectl apply \
  -f kubernetes/employee-directory/
```

---

## Validate the deployment

Check the Namespace:

```bash
kubectl get namespace employee-directory
```

Check the Deployment:

```bash
kubectl get deployment \
  -n employee-directory
```

Check the ReplicaSet:

```bash
kubectl get replicasets \
  -n employee-directory
```

Check the Pod:

```bash
kubectl get pods \
  -n employee-directory \
  -o wide
```

Check the Service:

```bash
kubectl get service \
  -n employee-directory
```

Check the Service endpoints:

```bash
kubectl get endpoints \
  employee-directory \
  -n employee-directory
```

The endpoint output should contain the application Pod IP and port `8000`.

---

## Inspect application logs

List the Pod:

```bash
kubectl get pods \
  -n employee-directory
```

Read logs through the Deployment:

```bash
kubectl logs \
  deployment/employee-directory \
  -n employee-directory
```

Follow logs:

```bash
kubectl logs \
  --follow \
  deployment/employee-directory \
  -n employee-directory
```

---

## Perform an application update

Edit the image tag in `deployment.yaml`, for example:

```yaml
image: ghcr.io/karl-george/employee-directory:v1.0.1
```

Apply the Deployment:

```bash
kubectl apply \
  -f kubernetes/employee-directory/deployment.yaml
```

Watch the rollout:

```bash
kubectl rollout status \
  deployment/employee-directory \
  -n employee-directory \
  --timeout=180s
```

View the rollout history:

```bash
kubectl rollout history \
  deployment/employee-directory \
  -n employee-directory
```

---

## Roll back a failed update

Roll back to the previous Deployment revision:

```bash
kubectl rollout undo \
  deployment/employee-directory \
  -n employee-directory
```

Wait for the rollback:

```bash
kubectl rollout status \
  deployment/employee-directory \
  -n employee-directory \
  --timeout=180s
```

Inspect the resulting image:

```bash
kubectl get deployment \
  employee-directory \
  -n employee-directory \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

---

## Remove the application

Delete the application resources:

```bash
kubectl delete \
  -f kubernetes/employee-directory/service.yaml

kubectl delete \
  -f kubernetes/employee-directory/deployment.yaml

kubectl delete \
  -f kubernetes/employee-directory/namespace.yaml
```

Deleting the Namespace also removes namespaced resources contained within it.

Do not delete the application after normal validation unless deliberately
testing the removal procedure.

---

## Troubleshooting

### `ImagePullBackOff`

Inspect the Pod:

```bash
kubectl describe pod \
  -n employee-directory \
  -l app.kubernetes.io/name=employee-directory
```

Common causes include:

- the image name is incorrect;
- the tag does not exist;
- the registry package is private;
- the node cannot reach the registry;
- registry credentials have not been configured.

### `CrashLoopBackOff`

Read the container logs:

```bash
kubectl logs \
  deployment/employee-directory \
  -n employee-directory
```

Inspect Pod events:

```bash
kubectl describe pod \
  -n employee-directory \
  -l app.kubernetes.io/name=employee-directory
```

Common causes include:

- an invalid application command;
- missing Python dependencies;
- missing configuration;
- database initialization errors;
- the application binding to the wrong address or port.

The application must listen on:

```text
0.0.0.0:8000
```

inside the container. Binding only to `127.0.0.1` prevents the Service from
reaching it through the Pod network.

### Service has no endpoints

Check the Service selector:

```bash
kubectl get service \
  employee-directory \
  -n employee-directory \
  -o yaml
```

Check the Pod labels:

```bash
kubectl get pods \
  -n employee-directory \
  --show-labels
```

Check the Service endpoints:

```bash
kubectl get endpoints \
  employee-directory \
  -n employee-directory
```

If the endpoint list is empty, the Service selector does not match the Pod
labels or the Pod has not become available.

### Application connection refused

Confirm that the application is listening on port `8000` inside the Pod:

```bash
kubectl logs \
  deployment/employee-directory \
  -n employee-directory
```

Check that the Service target port resolves to the Deployment's named `http`
port.

### Pod remains Pending

Describe the Pod:

```bash
kubectl describe pod \
  -n employee-directory \
  -l app.kubernetes.io/name=employee-directory
```

Review the Events section for:

- insufficient node resources;
- image pull problems;
- scheduling restrictions;
- volume problems.
