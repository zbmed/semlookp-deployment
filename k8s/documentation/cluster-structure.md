# K8s Namespace Structure and Labeling

This document describes the namespace organization and labeling conventions used in the external hosted Kubernetes clusters km-qa and km-prod.  

Namespaces are structured to isolate resources by project and application and are labeled accordingly to simplify:

- Resource usage tracking and management
- Cost attribution
- Security boundaries
---

## Namespace Overview

Each namespace represents a unique combination of **project** and **application**.  

| Namespace Name             | Project     | App            | Description                                      | Helm Chart                                                                                                                                       |
|-----------------------------|-------------|----------------|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| `nfdi4health-mda`           | nfdi4health | mda            | Metadata Annotation Service for NFDI4Health      | [App](https://github.com/nfdi4health/mda-deployment)                                                                                             |
| `nfdi4health-preview`       | nfdi4health | preview        | PreVIEW for NFDI4Health                          | [UI](https://github.com/nfdi4health/preview-deployment)                                                                                          |
| `nfdi4health-gateway`       | nfdi4health | gateway        | Gateway/middleware layer for the NFDI4Health CSH | [Gateway](https://github.com/zbmed/semlookp-deployment/tree/main/k8s/ols4-csh-gateway)                                                           |
| `zbmed-ts-health`           | zbmed       | ts-health      | Terminology Service for NFDI4Health under ZB MED | [UI](https://github.com/zbmed/semlookp-deployment/tree/main/k8s/semlookp-ui-health), [Backend](https://github.com/zbmed/ols4-backend-deployment) |
| `zbmed-ts-berd`             | zbmed       | ts-berd        | Terminology Service for BERD under ZB MED        | [UI](https://github.com/zbmed/semlookp-deployment/tree/main/k8s/semlookp-ui-berd), [Backend](https://github.com/zbmed/ols4-backend-deployment)                                                              |
| `zbmed-ts-nfdi4cat`         | zbmed       | ts-nfdi4cat    | Terminology Service for NFDI4Cat under ZB MED    | [App](https://github.com/zbmed/skosmos-deployment)                                                                                               |
| `ts4nfdi-portal`            | ts4nfdi     | portal         | TS4NFDI Service Portal                           | [UI](https://github.com/ts4nfdi/service-portal-deployment)                                                                                       |
| `ts4nfdi-gateway`           | ts4nfdi     | gateway        | TS4NFDI API-Gateway                              | [Gateway](https://github.com/ts4nfdi/api-gateway-deployment)                                                                                     |

---

## Namespace Definitions

Each namespace is defined as a standard Kubernetes `Namespace` resource with appropriate labels to track ownership and function:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nfdi4health-mda
  labels:
    project: nfdi4health
    app: mda
```

## Labeling Conventions

All objects (Deployments, Services, Pods, etc.) within each namespace follow a consistent labeling scheme, 
with k8s labels and private labels:

| Label Key | Example Value | Purpose                                   |
|------------|----------------|-------------------------------------------|
| `project` | `zbmed` | Identifies the owning project             |
| `app.kubernetes.io/instance` | `{{ .Release.Name }}` | Helm release instance name                |
| `app.kubernetes.io/part-of` | `ts-health` | Associated subsystem or service group/app |
| `app.kubernetes.io/component` | `ui` | Component type (e.g. ui, api, db)         |
| `app.kubernetes.io/managed-by` | `Helm` | Deployment management tool                |

> **Note:** These labels adhere to the [Kubernetes recommended labeling conventions](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/).

## Image Pull Secrets

GitHub container images are accessed via image pull secrets configured within each Helm chart or deployment script.
The process for creating and updating these secrets is described in:

📄 creating-image-pull-secrets.md

Ensure that each namespace has access to the appropriate secret before deploying new services.

