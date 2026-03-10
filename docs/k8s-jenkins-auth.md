# Jenkins Kubernetes RBAC Setup

This document explains how Jenkins is granted access to deploy applications into a Kubernetes cluster using a **ServiceAccount**, **RBAC Role**, and a **dedicated kubeconfig file**.

The setup ensures Jenkins interacts with Kubernetes using **least-privilege permissions** within the `project` namespace.

---

# 1. Create Jenkins ServiceAccount and RBAC Permissions

Apply the following Kubernetes manifest to create the Jenkins service account and assign it permissions inside the `project` namespace.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: project
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: project
rules:
  - apiGroups: ["", "apps", "batch", "extensions", "autoscaling"]
    resources:
      - pods
      - pods/log
      - services
      - endpoints
      - configmaps
      - secrets
      - deployments
      - replicasets
      - daemonsets
      - statefulsets
      - jobs
      - cronjobs
      - ingresses
      - persistentvolumeclaims
      - events
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: project
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: project
```

Apply the configuration:

```bash
kubectl apply -f jenkins-rbac.yaml
```

---

# 2. Generate Jenkins Kubernetes Token

Generate a token for the Jenkins ServiceAccount:

```bash
TOKEN="$(kubectl -n project create token jenkins --duration=24h)"
```

This token will allow Jenkins to authenticate with the Kubernetes API server.

---

# 3. Generate Jenkins kubeconfig

Create a kubeconfig file that Jenkins will use to connect to Kubernetes.

```bash
cat > config.yaml <<EOF
apiVersion: v1
kind: Config
clusters:
- name: k8s
  cluster:
    server: https://10.0.1.18:6443
    insecure-skip-tls-verify: true
contexts:
- name: jenkins
  context:
    cluster: k8s
    user: jenkins
    namespace: project
current-context: jenkins
users:
- name: jenkins
  user:
    token: ${TOKEN}
EOF
```

---

# 4. Verify Kubernetes Access

Test access using the generated kubeconfig:

```bash
kubectl --kubeconfig=config.yaml get pods
```

If successful, Jenkins will be able to interact with resources inside the `project` namespace.

---

# 5. Jenkins Integration

Upload the generated `config.yaml` to Jenkins as a **Secret File Credential**.

Example usage inside a Jenkins pipeline:

```bash
kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml
```

This allows Jenkins to deploy Kubernetes manifests securely.

---

# Security Best Practices

This setup follows several security practices:

- Namespace-scoped RBAC permissions
- Dedicated ServiceAccount for Jenkins
- Token-based authentication
- No cluster-admin privileges

For production environments consider:

- Using **OIDC authentication**
- Using **Vault for dynamic secrets**
- Enabling **TLS certificate verification instead of `insecure-skip-tls-verify`**

---
