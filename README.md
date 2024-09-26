---

# ArgoCD Setup and Application Deployment

## 1. Install ArgoCD

Run the following command to install ArgoCD:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 2. Ports for ArgoCD Server

The `argocd-server` listens on the following ports:
- **80**: HTTP
- **443**: HTTPS

## 3. Access the ArgoCD UI

Convert the ArgoCD Server service from type `ClusterIP` to `NodePort` and specify the node port for HTTPS:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports": [{"port": 443, "targetPort": 443, "nodePort": 32766}]}}}'
```

Access the UI at `https://<node-ip>:32766`

## 4. Get Initial Admin Password

Run the following command to decode the initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

## 5. Install ArgoCD CLI

Run the following commands to install the ArgoCD CLI:

```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

## 6. Create an ArgoCD Application

Using the ArgoCD CLI, create an application that deploys a three-tier application from a GitHub repository containing the YAML manifests:

```bash
argocd app create <app-name> \
  --repo <repository-url> \
  --path <path-to-yaml> \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace <namespace>
```

Sync the application:

```bash
argocd app sync <app-name>
```

Verify that the service is running in your Kubernetes cluster:

```bash
kubectl get svc -n <namespace>
```

## 7. Managing Secrets

To save secrets that need to be used in the Kubernetes cluster outside of the GitHub repository, consider using Kubernetes Secrets. Create a secret using the following command:

```bash
kubectl create secret generic <secret-name> --from-literal=<key>=<value> -n <namespace>
```

---

