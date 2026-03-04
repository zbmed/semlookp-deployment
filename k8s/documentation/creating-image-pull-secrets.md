# Using a GitHub Personal Access Token (PAT) for Pulling Images from GHCR in Kubernetes

This guide explains how to configure Kubernetes to pull container images from GitHub Container Registry (GHCR) using your GitHub account’s Personal Access Token (PAT).  
**Note:** These steps must be done **per namespace**, since secrets and service accounts in Kubernetes are namespace-scoped.

## Steps

1. **Create a Personal Access Token (PAT) on GitHub**

   Generate a new Personal Access Token in your GitHub account with at least the `read:packages` permission.

2. **Encode Username and Token in Base64**
   ```bash
   echo -n "$USERNAME:$TOKEN" | base64
3. **Create a .dockerconfig.json File**
```
{
        "auths": {
            "ghcr.io": {
                "auth": "$TOKEN"
            }
        }
    }
```
4. Create a Kubernetes Secret
```
kubectl create secret generic regcred \                                                                            
    --from-file=.dockerconfigjson=.dockerconfig.json \
    --type=kubernetes.io/dockerconfigjson
```
5. Set the Secret as Default for the Namespace
```
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "regcred"}]}'
```

This will automatically apply the secret to all pods that use the default service account in the given namespace.

6. Option 2: Reference the Secret in Deployment Manifests
Instead of modifying the service account, you can specify the image pull secret directly in your Kubernetes manifest:
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: ghcr.io/<your-username>/<your-image>:latest
      imagePullSecrets:
        - name: regcred
````
After completing either option, the namespace’s pods will be able to pull images from GitHub Container Registry using the authenticated configuration.