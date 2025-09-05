# Module 3 — Advanced Configuration Management in ArgoCD

> **Project:** Manage application configuration using Helm and Kustomize with ArgoCD, implement secrets management, and customize resource management & sync policies for safe, reproducible deployments.

---

## 📑 Table of Contents

* [Overview](#overview)
* [Objective](#objective)
* [Deliverables for Submission](#deliverables-for-submission)
* [Prerequisites](#prerequisites)
* [Repository Layout (Recommended)](#repository-layout-recommended)
* [Step-by-Step Implementation](#step-by-step-implementation)

  * [1) Create Cluster and Environment](#1-create-cluster-and-environment)
  * [2) Install ArgoCD and Tools](#2-install-argocd-and-tools)
  * [3) Prepare Helm Chart](#3-prepare-helm-chart)
  * [4) Prepare Kustomize Base & Overlays](#4-prepare-kustomize-base--overlays)
  * [5) Secrets Management Options](#5-secrets-management-options)
  * [6) Register Git Repo in ArgoCD & Create Apps](#6-register-git-repo-in-argocd--create-apps)
  * [7) Configure Resource Management & Sync Policies](#7-configure-resource-management--sync-policies)
  * [8) Validate and Test](#8-validate-and-test)
* [Common Troubleshooting](#common-troubleshooting)
* [Submission Checklist](#submission-checklist)
* [Appendix: Sample File Tree & Example Manifests](#appendix-sample-file-tree--example-manifests)
* [Author](#author)

---

## 📘 Overview

This project demonstrates advanced configuration management with ArgoCD by:

* Deploying an application using **Helm** and **Kustomize** stored in a Git repository.
* Managing secrets securely (SealedSecrets or SOPS/age).
* Customizing resource-difference ignore rules and sync policies.
* Capturing artifacts and evidence for submission.

---

## 🎯 Objective

By the end you will have:

* A Git repository containing a Helm chart and Kustomize configurations.
* ArgoCD applications that point to these configs and properly manage sync policies.
* Secrets stored/encrypted and applied safely.
* A README and supporting artifacts ready for submission.

---
📷 *Image Placeholder: ArgoCD Dashboard showing Synced/Healthy*

---

## ⚙️ Prerequisites

* Git (>= 2.x)
* Docker
* `kubectl` (>= 1.24)
* `argocd` CLI
* `helm` (v3+)
* `kustomize`
* `kind` or `minikube` / Kubernetes cluster
* `kubeseal` (if using SealedSecrets) or `sops`

---

## 📂 Repository Layout (Recommended)

```bash
my-argocd-project/
├── README.md
├── helm/
│   └── my-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
├── kustomize/
│   ├── base/
│   │   ├── kustomization.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── overlays/
│       ├── dev/
│       │   └── kustomization.yaml
│       └── prod/
│           └── kustomization.yaml
├── argocd-apps/
│   ├── helm-app.yaml
│   └── kustomize-app.yaml
└── secrets/
    ├── sealedsecret-db.yaml
    └── sops-secrets.yaml
```

📷 *Image Placeholder: Repository structure screenshot from IDE*

---

## 🚀 Step-by-Step Implementation

### 1) Create Cluster and Environment

```bash
kind create cluster --name argocd-dev
# OR
minikube start --driver=docker
```

### 2) Install ArgoCD and Tools

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 9090:443
```

📷 *Image Placeholder: ArgoCD login page screenshot*

### 3) Prepare Helm Chart

```bash
cd helm
helm create my-app
helm template my-app
```

📷 *Image Placeholder: Helm chart directory tree in IDE*

### 4) Prepare Kustomize Base & Overlays

```bash
kubectl kustomize kustomize/base
```

📷 *Image Placeholder: Kustomize overlays (dev/prod) screenshot*

### 5) Secrets Management Options

* **Option A:** SealedSecrets (recommended)
* **Option B:** SOPS (advanced)

📷 *Image Placeholder: Example sealedsecret YAML screenshot*

### 6) Register Git Repo in ArgoCD & Create Apps

```bash
argocd repo add https://github.com/your-username/your-repo.git
```

Create Helm and Kustomize apps via CLI or YAML manifests.

📷 *Image Placeholder: ArgoCD Applications list screenshot*

### 7) Configure Resource Management & Sync Policies

Add `ignoreDifferences` and `syncPolicy` in `Application` manifests.

📷 *Image Placeholder: Application YAML in IDE*

### 8) Validate and Test

```bash
argocd app get helm-my-app
argocd app get kustomize-dev
kubectl get pods -n default
```

📷 *Image Placeholder: Running pods screenshot*

---

## 🛠 Common Troubleshooting

* `Pods CrashLoopBackOff` → Check logs.
* `OutOfSync` → Run `argocd app diff` and `argocd app sync`.
* Repo issues → Verify `argocd repo list`.
* Secrets not applied → Ensure SealedSecrets controller is running.

---

## ✅ Submission Checklist

* [ ] Git repo link
* [ ] README.md (this file)
* [ ] `helm/my-app` chart
* [ ] `kustomize/base` and overlays
* [ ] `argocd-apps/*.yaml`
* [ ] `secrets/` directory
* [ ] Screenshots (ArgoCD + deployed app)
* [ ] Notes on challenges faced

---

### Helm Application (argocd-apps/helm-app.yaml)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-username/your-repo.git'
    path: 'helm/my-app'
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### Kustomize Application (argocd-apps/kustomize-app.yaml)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kustomize-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-username/your-repo.git'
    path: 'kustomize/overlays/dev'
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  ignoreDifferences:
  - group: networking.k8s.io
    kind: Ingress
    jsonPointers:
    - /metadata/annotations
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

📷 *Image Placeholder: Final deployed application screenshot*
