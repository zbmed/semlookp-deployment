# Setup namespace cert manager

```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-prod
  namespace: zbmed-ts-health
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: bla@12.de
    privateKeySecretRef:
      name: account-key-prod
    solvers:
    - http01:
       ingress:
         ingressClassName: nginx
```