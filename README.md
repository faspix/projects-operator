# Kubernetes Projects Operator

An Ansible-based Kubernetes Operator to manage multi-environment projects with namespaces, resource quotas, RBAC, network policies, and service accounts. This operator allows declarative management of project environments using a Custom Resource Definition (CRD).

---

## Table of Contents
- [Features](#features)
- [Installation](#installation)
- [Example](#example)

---

## Features

- **Namespace management**: Automatically creates project-specific namespaces per environment.
- **Resource quotas & limits**: Configures `ResourceQuota` and `LimitRange` for each namespace.
- **RBAC setup**: Assigns roles to Admins, Developers, and Viewers via `RoleBinding`.
- **Network policies**: Supports default deny-all, DNS access, egress to the internet, and controlled ingress between namespaces.
- **Service accounts**: Automatically creates service accounts as specified in the project definition.
- **Custom Resource Definition (CRD)**: Project CRD defines the desired state of the project environments, including owner, environments, resources, RBAC, and network policies.

---

## Installation

1. **Clone projects-operator:**
```bash
git clone git@github.com:faspix/projects-operator.git
```

2. **Apply the CRD:**
```bash
kubectl apply -f config/crd/bases/projects.faspix.com.yaml
```

3. **Run the make target:**
```bash
make deploy IMG=ghcr.io/faspix/projects-operator/projects-operator:latest
```

4. **After it is built, test it on a cluster:**
```bash
kubectl apply -f config/samples/v1alpha1_project.yaml
```
---
## Examples

### Minimal Example
For a simple project with a single environment and basic RBAC:
```yaml
apiVersion: faspix.com/v1alpha1
kind: Project
metadata:
  name: simple-project
spec:
  owner: "my-team"
  environments:
    - name: dev
      resources:
        quota:
          pods: 10
          requests:
            cpu: "1"
            memory: "2Gi"
  rbac:
    admingroup: "my-team-admins"
```
**Apply the minimal project:**
```bash
kubectl apply -f simple-project.yaml
```
---

### Example

Below is an example of a `Project` custom resource that defines a multi-environment setup for a development team. This configuration provisions namespaces, resource quotas, RBAC, network policies, and service accounts for `prod`, `stage`, and `dev` environments.

### Example Project Custom Resource

**Metadata and Ownership**
The `metadata` section defines the name and labels for the project, while `spec.owner` specifies the team owning the project.
```yaml
apiVersion: faspix.com/v1alpha1
kind: Project
metadata:
  labels:
    app.kubernetes.io/name: operator-k8s
    app.kubernetes.io/managed-by: kustomize
  name: project-sample
spec:
  owner: "dev-team-1" # Team responsible for this project
```
**Environments**
The `environments` section defines the namespaces (e.g., `prod`, `stage`, `dev`) and their configurations, including network policies, resource quotas, and pod security settings.
```yaml
  environments:
    - name: prod # Production environment
      networkpolicies:
        denyall: true # Deny all inbound traffic by default
        allowdns: false # Disable DNS access
        allowegressinternet: false # Disable internet egress 
        allowingressfromsamenamespace: true # Allow ingress within the same namespace
        allowingressfromothernamespaces: true # Allow ingress from other namespaces
      finalizers:
        - kubernetes # Ensure cleanup on deletion
      serviceaccounts:
        - deploy # Service account for CI/CD
        - monitoring # Service account for monitoring
      podsecurity:
        type: audit # Pod security policy type
        level: baseline # Security level
        version: latest # Use the latest version
      resources:
        quota: # Resource quotas for the namespace
          pods: 20 # integer
          configmaps: 20 # integer
          secrets: 20 # integer
          storage: "100Gi"
          requests:
            cpu: "5"
            memory: "50Gi"
          limits:
            cpu: "8"
            memory: "80Gi"
        defaultlimitrange: # Default resource limits for pods
          requests:
            cpu: "1"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "1Gi"
          max:
            cpu: "1"
            memory: "1Gi"
          min:
            cpu: "100m"
            memory: "128Mi"
    - name: stage # Staging environment
      resources:
        defaultlimitrange:
          requests:
            cpu: "0.5"
            memory: "1Gi"
          limits:
            cpu: "0.8"
            memory: "1Gi"
    - name: dev # Development environment
```
**RBAC Configuration**
The `rbac` section assigns roles to groups for administrative, developer, and viewer access.
```yaml
  rbac: # Role-based access control
    admingroup: "dev-team-1-admins" # Admin group with full access
    devgroup: "dev-team-1-devs" # Developer group with edit access
    viewergroup: "dev-team-1-viewers" # Viewer group with read-only access
```
**Apply the project:**
Save the above configuration to a file (e.g., my-project.yaml) and apply it to your cluster:
```bash
kubectl apply -f my-project.yaml
```

After applying the project, verify the created resources:
```bash
kubectl get namespaces
kubectl get resourcequotas -n project-sample-prod
kubectl get rolebindings -n project-sample-prod
kubectl get networkpolicies -n project-sample-prod
```
Expected output includes namespaces like project-sample-prod, project-sample-stage, and project-sample-dev, along with their respective quotas, role bindings, and network policies.