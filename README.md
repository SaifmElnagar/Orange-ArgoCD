### 1. **Install ArgoCD**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. **Ports ArgoCD Server Listens On**
The `argocd-server` listens on the following ports:
- **HTTP**: 8080
- **HTTPS**: 443

### 3. **Access the ArgoCD UI by Changing Service Type to NodePort**
To access the ArgoCD UI, update the `argocd-server` service to use NodePort and assign a custom port (32766) for HTTPS.

Edit the ArgoCD server service:
```bash
kubectl -n argocd edit svc argocd-server
```

Change the `type` from `ClusterIP` to `NodePort`, and add the following line under `ports` for HTTPS:
```yaml
type: NodePort
ports:
  - name: https
    port: 443
    targetPort: 8080
    nodePort: 32766  # Assign node port 32766 for HTTPS
```

Save the file, and you will now be able to access the ArgoCD UI at:
```
https://<NodeIP>:32766
```

### 4. **Decode Initial Admin Password**
By default, the ArgoCD admin password is stored as a base64-encoded secret. To decode and retrieve it, use this command:
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o json | jq .data.password -r | base64 -d
```

### 5. **Install ArgoCD CLI**
To install the ArgoCD CLI on Linux, run the following commands:
```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

### 6. **Create an ArgoCD Application**
To create an ArgoCD application that deploys a three-tier application from a GitHub repository, follow these steps:

1. **Login to ArgoCD CLI**
```bash
argocd login <NodeIP>:32766 --insecure
```

2. **Create the Application**
```bash
argocd app create three-tier-app \
--repo https://github.com/<your-repo>/<three-tier-app-repo>.git \
--path <path-to-yaml-files> \
--dest-server https://kubernetes.default.svc \
--dest-namespace default
```

3. **Sync the Application**
```bash
argocd app sync three-tier-app
```

4. **Verify the Application**
To check if the application is deployed and running in your Kubernetes cluster:
```bash
kubectl get all -n default
```
