## Kubernetes-Native AI Model Deployment Platform

A Kubernetes-native platform for dynamically deploying and interacting with GGUF-based LLMs using GitOps workflows, ArgoCD, and `llama.cpp`.

This project provides:

* Dynamic model deployment
* GitOps-based infrastructure reconciliation
* Runtime inference APIs
* Kubernetes-native service discovery
* ArgoCD-powered continuous deployment
* Ingress-based external routing
* React-based inference dashboard
* Automatic manifest generation for deployed models

---

## Architecture

```text
User
  ↓
Frontend (React)
  ↓
Ingress NGINX
  ├── /        → Frontend Service
  └── /api     → Backend Service
                     ↓
               GitOps Orchestrator
                     ↓
              Cluster Config Repo
                     ↓
                  ArgoCD
                     ↓
         Dynamically Generated Deployments
                     ↓
            llama.cpp Inference Pods
```

---

## Features

### Dynamic Model Deployment

Upload a GGUF model URL through the platform API.

The backend automatically:

1. Generates Kubernetes manifests
2. Updates kustomization files
3. Commits changes to GitHub
4. Pushes changes to the GitOps repository
5. ArgoCD reconciles the cluster state
6. New inference pod becomes available

---

### GitOps Workflow

This platform follows a GitOps deployment model.

#### [Application Repository](https://github.com/thecrusader25225/ml-deployment)

Contains:

* Frontend source
* Backend source
* Github Actions CI pipelines

#### [Cluster Configuration Repository](https://github.com/thecrusader25225/cluster-config)

Contains:

* Kubernetes manifests
* Kustomize overlays
* Generated inference deployments
* Ingress configuration

The backend dynamically updates the cluster configuration repository.

---

### Runtime Inference

Each model is exposed internally through Kubernetes services.

Example:

```text
tinyllama-chat.inference-dev.svc.cluster.local
```

Inference requests are routed through the backend API.

---

### Frontend Dashboard

The frontend provides:

* Available model list
* Chat interface per model
* Persistent local chat history
* Dynamic inference interaction
* API endpoint visibility

---

### Tech Stack

| Category | Technologies |
|---|---|
| Frontend | React, Vite |
| Backend | Node.js, Express |
| AI / Inference | llama.cpp, Python |
| Containerization | Docker |
| Orchestration | Kubernetes |
| GitOps / CD | ArgoCD |
| CI | GitHub Actions |
| Networking | Ingress NGINX |
| Configuration | Kustomize |
| Infrastructure | Google Cloud Compute Engine |
---

## Repository Structure

```text
.
├── frontend/
├── backend/
├── .github/workflows/
└── cluster-config/
    ├── apps/
    │   ├── platform/
    │   └── inference/
    └── environments/
```

---

## CI/CD Flow

### Frontend / Backend CI

On push:

1. Build Docker image
2. Push image to Docker Hub
3. Update kustomization image tag
4. Push changes to cluster-config repository
5. ArgoCD syncs cluster automatically

---

## Example Model Deployment

### Register Model

```bash
curl -X POST http://<platform-url>/api/models \
  -H "Content-Type: application/json" \
  -d '{
    "name":"tinyllama-chat",
    "modelUrl":"https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/tinyllama-1.1b-chat-v1.0.Q2_K.gguf",
    "runtime":"llama.cpp"
  }'
```

---

### Inference Request

```bash
curl -X POST http://<platform-url>/api/models/tinyllama-chat/infer \
  -H "Content-Type: application/json" \
  -d '{
    "prompt":"What is Kubernetes?",
    "n_predict":64
  }'
```

---

## Kubernetes Components

### Platform Namespace

Contains:

* Frontend deployment
* Backend deployment
* Ingress
* Platform services

---

### Inference Namespace

Contains dynamically generated:

* Model deployments
* Model services
* Runtime containers

---

## Ingress Routing

```text
/api  → backend
/     → frontend
```

Ingress uses host-based routing with `sslip.io`.

---

## Current Limitations

* Models are not yet persisted across backend restarts
* GGUF files currently download during deployment
* External DNS currently uses dynamic VM IPs
* ArgoCD currently relies on polling instead of webhooks

---

## Planned Improvements

* Persistent volumes for GGUF storage
* SQLite metadata persistence
* ArgoCD webhook-based instant sync
* Authentication and RBAC
* Streaming inference responses
* GPU scheduling support
* Multi-runtime support
* Deployment health monitoring
* Canary deployments
* HTTPS with custom domain

---

## Running the Platform

### Start Kubernetes Cluster

```bash
kubectl get nodes
```

---

### Deploy ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

### Apply Applications

```bash
kubectl apply -f applications/
```

---

### Access Platform

```text
http://<external-ip>.sslip.io:<nodeport>
```

---

## License

[MIT](LICENSE)
