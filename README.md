# Cluster Config – GitOps Configuration for <a href="https://github.com/thecrusader25225/webapp-services">Webapp Services</a>

This repository contains the **Kubernetes and Argo CD configuration** for deploying and operating the *Webapp Services* application using a **GitOps workflow**.

It defines the **desired state of the cluster**, including:
- application deployments
- environment separation
- ingress configuration
- autoscaling and health checks
- Argo CD Applications (app-of-apps pattern)

This repository does **not** contain application source code.

---

## Purpose

The purpose of this repository is to act as the **single source of truth for cluster state**.

All changes to:
- what runs in the cluster
- where it runs (dev / prod)
- how it is exposed
- how it scales and heals  

are made **declaratively via Git** and reconciled by **Argo CD**.

No workloads are modified directly via `kubectl`.

---

## Cluster Topology

The configuration targets a **self-managed Kubernetes homelab cluster** with:

- Provisioned using **kubeadm**
- CNI: **Calico**
- **1 control-plane node**
- **2 worker nodes**

---

## Repository Structure

```
cluster-config/
├───apps/
│   ├───backend/
│   │   ├───base/
│   │   └───overlays/
│   │       ├───dev/
│   │       └───prod/
│   └───frontend/
│       ├───base/
│       └───overlays/
│           ├───dev/
│           └───prod/
├───argocd/
│   ├───dev/
│   └───prod/
└───environments/
    ├───base/
    └───overlays/
        ├───dev/
        └───prod/
```


---

## Environment Separation

Environments are isolated using **Kustomize overlays** and **namespaces**:

- `dev`
- `prod`

Each environment has:
- independent overlays
- separate ingress configuration
- explicit environment identity injected via configuration

No manifests are shared implicitly between environments.

---

## Argo CD Setup

This repository uses the **app-of-apps pattern**.

### Root Applications

- `dev-root`
- `prod-root`

Each root application:
- lives in the `argocd/` directory
- references child applications declaratively
- manages synchronization behavior for its environment

### Sync Strategy

- **Development**
  - Auto-sync enabled
  - Automatic pruning
- **Production**
  - Manual sync
  - Explicit promotion required

This enforces fast iteration in dev and controlled releases in prod.

---

## Image Promotion Flow

This repository does **not build images**.

Instead:
- CI pipelines in the application repository build and publish images
- Image tags are updated in **environment overlays** here
- Argo CD reconciles the new desired state

This preserves a strict separation between:
- **artifact creation** (CI)
- **artifact deployment** (GitOps)

---

## Platform & Runtime Characteristics

The following Kubernetes primitives are configured as baseline runtime behavior:

- **Health Probes**
  - `livenessProbe` and `readinessProbe` are defined for workloads
  - Used to validate pod health and traffic readiness

- **Horizontal Pod Autoscaler (HPA)**
  - Backend workloads are configured with HPA
  - Used to validate autoscaling mechanics rather than load testing

---

## What This Repository Does NOT Contain

- Application source code
- CI pipelines
- Build logic
- Dockerfiles

Those live in the application repository:<a href="https://github.com/thecrusader25225/webapp-services">webapp-services</a>

---

## Design Principles

- Git is the single source of truth
- No direct cluster mutation
- Clear environment boundaries
- Declarative over imperative
- Minimal but realistic configuration


