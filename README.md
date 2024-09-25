# Knative Tekton CI with GitHub, GitHub Actions Workflows
GitHub Actions Triggered Knative Tekton In-K8s Cluster CI Pipeline for Go App (Containerize, Aquasec Trivy NIST 800-53 CVE Scan)


## Prerequistes

- Kubernetes Cluster
- Helm

## Installation w/ out ArgoCD or FluxCD

The follow workflow is required to configure Tekton and Tetkon Pipeline with GitHub Actions.

- Install (Provision) Kubernetes Cluster
- Install (Provision) Tekton Operator to Kubernetes Cluster
- Provision Tekton RBAC Configuration for Kubernetes Cluster and Tekton Namespace
- Provision GitHub Actions WebHook to Trigger Tetkton CI Pipeline w/ Tekton Triggers

### Install (Provision) Kubernetes Cluster

Tekton requires a Kubernetes Cluster since it is deployed as a Kubernetes Operator Helm Chart.
Tetkon provides a set of CRDs as covererd in the latter part of this page.

### GCP GKE Cluster Provision w/ Crossplane

IN-PROGRESS

### Azure AKS Cluster Provision w/ Crossplane

IN-PROGRESS

### Install (Provision) Tekton to Kubernetes Cluster
To install to the Kubernetes Cluster with Helm, apply the following.

```shell
helm repo add tekton https://tekton.dev/charts/
helm repo update
helm install tekton-pipelines tekton/tekton-pipeline --namespace tekton-pipelines --create-namespace
```

### Provision Tekton RBAC Configuration for Kubernetes Cluster and Tekton Namespace

### Provision GitHub Actions WebHook to Trigger Tetkton CI Pipeline w/ Tekton Triggers



## Installation w/ ArgoCD

## Installation w/ FluxCD
