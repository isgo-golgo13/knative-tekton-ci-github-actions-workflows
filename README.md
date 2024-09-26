# Knative Tekton CI with GitHub Actions and Hashicorp Vault Operator Workflows
GitHub Actions Triggered Knative Tekton In-K8s Cluster CI Pipeline for Go App (Containerize, Aquasec Trivy NIST 800-53 CVE Scan).

The following Kubernetes services are part of this architectural workflow.

- Hashicorp Vault Kubernetes Operator
- External Secrets (Kubernetes) Operator
- Tekton Knative Operator for Parallel Workflows (used for CI and CD)

The required workflow result will provide a coordination of Hashicorp Vault Kubernetes Operator to serve as the
cloud-agnostic Kubernetes-Native (In-Cluster) `Secrets` store. The secret credentials required for Tekton to reference

## Prerequistes

- Kubernetes Cluster
- Helm

## Installation w/ out ArgoCD or FluxCD

The follow workflow is required to configure Tekton and Tetkon Pipeline with GitHub Actions.

- Install (Provision) Kubernetes Cluster
- Install (Provision) Tekton Operator to Kubernetes Cluster
- Install (Provision) Hashicorp Vault Kubernetes Operator to the Kubernetes Cluster
- Configure Hashicorp Vault Kubernetes Operator Auto-Unsealing Workflow (Kubernetes Job)
- Install Kubernetes External Secrets Operator (ESO) and Register ESO w/ Hashicorp Vault
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

## Install (Provision) Hashicorp Vault Kubernetes Operator to the Kubernetes Cluster

The GitOps declarative configuration for Vault requires a two stepped actions.

- Auto-Start (does a `vault init`)
- Auto-Unsealing (does a `vault unseal`)

To provide this declarative configuraiton the following `.values-vault-operator.yaml` configuration is required
and is forward to the Helm install for Vault.

```yaml
# vault-values-operator.yaml for Vault Helm chart with auto-unseal
server:
  extraEnvironmentVars:
    VAULT_SEAL_KUBERNETES_SECRET: "true"
  dataStorage:
    enabled: true
  ha:
    enabled: true
  unsealConfig:
    enabled: true
    kubernetes:
      secretName: vault-unseal-keys
      namespace: vault
      keyPrefix: vault-unseal-key
```
To install Hashicorl Vault Kubernetes Operator apply the following. **NOTE** In full-GitOps auto-provisioning, a
direct `helm repo update` and `helm install or helm upgrade -i vault hashicorp/vault -n vault` is deferred to ArgoCD using
`GitOps for ArgoCD` design pattern and ArgoCD App-of-Apps or newer ArgoCD `ApplicationSets` or FluxCD.

```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault --namespace vault --create-namespace
```


### Provision Tekton RBAC Configuration for Kubernetes Cluster and Tekton Namespace

### Provision GitHub Actions WebHook to Trigger Tetkton CI Pipeline w/ Tekton Triggers



## Installation w/ ArgoCD

## Installation w/ FluxCD
