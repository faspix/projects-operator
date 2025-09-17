# Kubernetes Projects Operator

An Ansible-based Kubernetes Operator to manage multi-environment projects with namespaces, resource quotas, RBAC, network policies, and service accounts. This operator allows declarative management of project environments using a Custom Resource Definition (CRD).

---

## Features

- **Namespace management**: Automatically creates project-specific namespaces per environment.
- **Resource quotas & limits**: Configures `ResourceQuota` and `LimitRange` for each namespace.
- **RBAC setup**: Assigns roles to Admins, Developers, and Viewers via `RoleBinding`.
- **Network policies**: Supports default deny-all, DNS access, egress to the internet, and controlled ingress between namespaces.
- **Service accounts**: Automatically creates service accounts as specified in the project definition.
- **Custom Resource Definition (CRD)**: Project CRD defines the desired state of the project environments, including owner, environments, resources, RBAC, and network policies.

## Installation



1. **Clone the awx-resource-operator**

```bash
git clone git@github.com:faspix/projects-operator.git
```

2. **Log in to your cluster.**

```bash
kubectl login <cluster-url>
```

3. **Run the make target**

```bash
IMG=projects-operator:latest make deploy
```


