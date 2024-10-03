

---

### Introduction

In a Kubernetes (K8S) cluster, secrets are essential for storing sensitive information like passwords, API keys, TLS certificates, and other credentials used by applications. However, storing these secrets directly in source code repositories like GitHub poses significant security risks. If such repositories are compromised, the exposed secrets could lead to unauthorized access and data breaches.

This report outlines best practices for securely managing and saving secrets for Kubernetes clusters while keeping them out of GitHub repositories.

---

### Risks of Storing Secrets in GitHub Repositories

1. **Public Access**: Even in private repositories, there is always a risk that sensitive data could become publicly accessible due to configuration errors or access misuse.
2. **Version Control**: Secrets are stored as part of the repository's history, making it difficult to remove sensitive data once committed.
3. **Accidental Exposure**: Developers can accidentally push sensitive information to the repository without noticing, exposing credentials to a wider audience.

---

### Best Practices for Storing Secrets Outside of GitHub Repositories

#### 1. **Use Kubernetes Secrets**

Kubernetes has a built-in mechanism for managing sensitive data called Secrets. This object type can store sensitive information in base64-encoded format, but you should avoid hardcoding these in your manifest files.

- **Create a Secret**:
```bash
kubectl create secret generic db-credentials --from-literal=username=myuser --from-literal=password=mypassword
```

- **Use the Secret in a Pod**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: myapp-container
    image: myapp
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

However, K8S Secrets are only base64-encoded, not encrypted. For sensitive data, you should also look into further encryption and management tools.

---

#### 2. **Use Kubernetes External Secrets**

**Kubernetes External Secrets** integrates Kubernetes with external secret management systems such as AWS Secrets Manager, HashiCorp Vault, or Google Cloud Secrets. With this, the secret data can be pulled from external secret stores and injected directly into the Kubernetes cluster, avoiding storing secrets in the cluster itself.

- **How External Secrets Work**:
  - External Secrets sync secrets from external services (e.g., AWS Secrets Manager) to Kubernetes Secrets.
  - You define a secret in an external manager, and Kubernetes retrieves the secret on demand.

- **Example using AWS Secrets Manager**:
  1. Install the External Secrets operator:
  ```bash
  kubectl apply -f https://github.com/external-secrets/kubernetes-external-secrets/releases/download/v0.3.0/kubernetes-external-secrets.yaml
  ```

  2. Create an External Secret that syncs data from AWS Secrets Manager:
  ```yaml
  apiVersion: 'kubernetes-client.io/v1'
  kind: ExternalSecret
  metadata:
    name: my-secret
  spec:
    backendType: secretsManager
    data:
      - key: prod/db/password
        name: password
  ```

  This example retrieves the secret stored at `prod/db/password` in AWS Secrets Manager and injects it into Kubernetes as a secret named `my-secret`.

---

#### 3. **Use HashiCorp Vault**

**HashiCorp Vault** is a popular tool for securely managing secrets. Vault allows you to dynamically manage, secure, and control access to tokens, passwords, and other secrets across different environments.

- **Benefits of HashiCorp Vault**:
  - Centralized storage and control over secrets.
  - Fine-grained access control and audit logging.
  - Support for dynamic secrets, such as database credentials that are generated on-demand.

- **How to Integrate Vault with Kubernetes**:
  - Install the **Vault Helm Chart** or **Vault Agent Injector** to your cluster.
  - Use a Vault agent to inject secrets into your pods as environment variables or files, avoiding the need to store them in Kubernetes Secrets.

  Example workflow:
  - Vault stores a secret.
  - A Vault Agent sidecar container fetches the secret and injects it into a pod.
  - The application accesses the secret via environment variables or mounted files.

---

#### 4. **Use Helm Secrets Plugin**

When deploying applications using Helm charts, sensitive values can easily end up in plain text in `values.yaml` files. To avoid this, you can use the **Helm Secrets plugin**, which encrypts the secrets using GnuPG.

- **How to Use Helm Secrets**:
  1. Install the plugin:
  ```bash
  helm plugin install https://github.com/jkroepke/helm-secrets
  ```

  2. Encrypt sensitive data in `values.yaml` using GnuPG:
  ```bash
  sops -e values.yaml > secrets.yaml
  ```

  3. Deploy the encrypted secrets:
  ```bash
  helm secrets install my-release -f secrets.yaml
  ```

This ensures that even if someone accesses the `values.yaml` file, the secrets remain encrypted.

---

#### 5. **Environment Variables and ConfigMaps**

Another secure method of providing secrets to applications is using environment variables or ConfigMaps, combined with a CI/CD pipeline (e.g., Jenkins, GitHub Actions) to inject the secrets at deployment time. This can be done by using secret management systems and injecting them into the cluster dynamically.

For example, in Jenkins:
- Store sensitive data in Jenkins credentials.
- Access them during the CI/CD pipeline and inject into Kubernetes manifests or Helm charts during the deployment process.

---

### Conclusion

Secrets should never be stored in a GitHub repository, as this exposes sensitive information to unintended users. By using external secret management tools, such as Kubernetes Secrets, External Secrets, Vault, and Helm Secrets, we can securely store and manage secrets outside of the repository. These methods help reduce the attack surface, ensure fine-grained access control, and improve security across the deployment pipeline.

The best approach depends on the specific requirements of your system, such as the level of security needed, integration complexity, and compliance regulations. For most organizations, a combination of Kubernetes Secrets and an external secret management system (e.g., AWS Secrets Manager or Vault) will provide the optimal balance between security and operational complexity.
