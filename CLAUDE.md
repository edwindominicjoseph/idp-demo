# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Internal Developer Platform (IDP) demo** — a homelab GitOps environment built on top of Google's [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) sample e-commerce application. The goal is to demonstrate a full GitOps delivery pipeline using Jenkins, Argo CD, and Kustomize on a single-node Kubernetes cluster.

## Architecture

### Deployment flow

```
GitHub Actions (image build) → push image tag → Jenkins pipeline triggered
→ Manual approval (prod only) → update kustomization.yaml image tag in Git
→ Argo CD detects Git change → syncs to Kubernetes cluster
→ Smoke tests → Rollback gate (Prometheus error-rate watch)
```

### Directory layout

| Path | Purpose |
|------|---------|
| `src/` | Upstream microservice source code (Go, Java, C#, Node.js, Python) |
| `kubernetes-manifests/` | Raw manifests for use with `skaffold` only — **not directly deployable** |
| `kustomize/` | Kustomize base + optional feature components for cluster deployment |
| `gitops/environments/staging/` | Staging kustomization overlay (image tags updated by Jenkins) |
| `gitops/environments/prod/` | Prod kustomization overlay (image tags updated by Jenkins) |
| `gitops/infra/argo-cd/` | Argo CD Helm values (`argocd-values.yaml`) |
| `gitops/infra/jenkins/` | Jenkins Helm values (`jenkins-values.yaml`) |
| `jenkinsfiles/` | Jenkins declarative pipelines (`Jenkinsfile.staging`, `Jenkinsfile.prod`) |
| `apps/boutique/` | App-specific assets (Dockerfile currently a stub — see CHG-PROD-2026-006) |

### Microservices (in `src/`)

Each service has its own language/runtime:

| Service | Language |
|---------|----------|
| frontend | Go |
| checkoutservice | Go |
| productcatalogservice | Go |
| shippingservice | Go |
| recommendationservice | Python |
| adservice | Java (Gradle) |
| cartservice | C# (.NET) |
| currencyservice | Node.js |
| paymentservice | Node.js |
| emailservice | Python |
| loadgenerator | Python (Locust) |
| shoppingassistantservice | Python |

### Kustomize components (in `kustomize/components/`)

Optional overlays that can be composed into a deployment: `cymbal-branding`, `google-cloud-operations`, `memorystore`, `spanner`, `alloydb`, `network-policies`, `service-mesh-istio`, `non-public-frontend`, `single-shared-session`, `without-loadgenerator`, and image tag/registry overrides.

## Deployment Commands

### Deploy with Kustomize (default base)

```bash
cd kustomize/
kubectl apply -k .
```

### Preview manifests without deploying

```bash
kubectl kustomize kustomize/
```

### Add a Kustomize component

```bash
cd kustomize/
kustomize edit add component components/cymbal-branding
```

### Check pod status

```bash
kubectl get pods
kubectl get service frontend-external | awk '{print $4}'
```

## Jenkins Pipelines

Both pipelines run on a `homelab-agent` pod with `alpine/k8s:1.29.2` (tools) + `maven:3.9-eclipse-temurin-17` containers.

- **`Jenkinsfile.staging`** — triggered on push; validates tools and performs Git clone. Argo CD sync and smoke tests are placeholders (Phase 3/5).
- **`Jenkinsfile.prod`** — requires manual approval; records rollback tag from current `gitops/environments/prod/kustomization.yaml` before deploy. Update-image-tag, Argo CD sync, smoke tests, and Prometheus rollback gate are placeholders pending Phase 3/5.

Required Jenkins credentials: `github-token` (username+password), `github-token-status` (secret text).

## Infrastructure Helm Values

- **Argo CD** (`gitops/infra/argo-cd/argocd-values.yaml`): NodePort service, insecure mode (TLS offloaded to Tailscale), homelab-tuned resource limits.
- **Jenkins** (`gitops/infra/jenkins/jenkins-values.yaml`): ClusterIP service, JCasC configures the Kubernetes cloud with per-tool containers (kubectl, argocd, curl, maven, docker-dind), 5 Gi persistence, RBAC for pod creation.

## Build Notes

- **Container image builds are owned by GitHub Actions**, not Jenkins. Jenkins only orchestrates deployment.
- The `apps/boutique/Dockerfile` is currently a stub (`nginx:alpine`). Real Dockerfile is planned for CHG-PROD-2026-006.
- `kubernetes-manifests/` requires `skaffold` to inject correct image tags — do not `kubectl apply` these directly; use `kustomize/` instead.
