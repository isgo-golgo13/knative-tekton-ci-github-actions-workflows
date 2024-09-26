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
Tekton provides a set of CRDs as covererd in the latter part of this page.

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

- Auto-Init (does a `vault init`)
- Auto-Unsealing (does a `vault unseal`)

There is an Auto-Unseal feature of Vault. This allows Vault to automatically unseal itself using a
Cloud provided Key Management Service (KMS) orKubernetes Secrets. This configuration can get applied declaratively in Vaults Helm chart.

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
Vault will now automatically initialize and unseal itself using Kubernetes secrets without needing manual intervention


To install Hashicorp Vault Kubernetes Operator apply the following. In full-GitOps auto-provisioning, a
direct `helm repo update` and `helm install` or `helm upgrade` is correctly deferred to ArgoCD using
`GitOps for ArgoCD` design pattern and ArgoCD App-of-Apps or newer ArgoCD `ApplicationSets` or FluxCD.

```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault --namespace vault --create-namespace -f vault-values-operator.yaml
```


To provide automatic configuration for Vault to authorize to Kubernetes auth engine apply the following.

```yaml
# vault-kubernetes-auth-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: vault-kubernetes-auth-job
  namespace: vault
spec:
  template:
    spec:
      serviceAccountName: vault
      containers:
      - name: vault
        image: vault:latest
        command: ["vault", "write"]
        args:
          - "auth/kubernetes/config"
          - "token_reviewer_jwt=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
          - "kubernetes_host=https://$KUBERNETES_PORT_443_TCP_ADDR:443"
      restartPolicy: OnFailure
```
This job is triggered automatically after Vault is deployed, declaratively configuring the Kubernetes auth.


Next what is required is to declaratively and automatically provide the Vault policy creation. This is configured
with a Kubernetes `Job` and Kubernetes `ConfigMap.

The ConfigMap for this.

```yaml
# vault-policy-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vault-policy
  namespace: vault
data:
  tekton-policy.hcl: |
    path "secret/data/dockerhub/*" {
      capabilities = ["read"]
    }
```

The Job for this.

```yaml
# vault-policy-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: vault-policy-job
  namespace: vault
spec:
  template:
    spec:
      serviceAccountName: vault
      containers:
      - name: vault
        image: vault:latest
        command: ["vault", "policy", "write", "tekton-policy", "/vault/policies/tekton-policy.hcl"]
        volumeMounts:
        - name: policy
          mountPath: /vault/policies
      volumes:
      - name: policy
        configMap:
          name: vault-policy
      restartPolicy: OnFailure
```


## Install Kubernetes External Secrets Operator (ESO) and Register ESO w/ Hashicorp Vault

To install the Kubernetes External Secrets Opertor to Kubernetes Cluster apply the following.

```shell
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install external-secrets external-secrets/external-secrets --namespace eso --create-namespace
```

Now a configuration for ESO is to dynamically pull the stored secrets in Hashicorp Vault at the point of
provisioning (point of the secrets pull). This registration of associating ESO with Hashicorp Vault is acheived with
the ESO `ClusterSecretStore` CR. The ClusterSecretStore CR will point to Hashicorp Vault. ESO can work with other secret store
services such as GKE KeyVault or AWS Secrets Manager.

The following is the required `ClusterSecretStore` CR that points to Hashicorp Vault.

```yaml
# eso-clustersecretstore.yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-secret-store
  namespace: eso
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "secret/data"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "auth/kubernetes"
          role: "tekton-sa"
```

And to apply the resource.

```shell
kubectl apply -f eso-clustersecretstore.yaml
```




### Provision Tekton RBAC Configuration for Kubernetes Cluster and Tekton Namespace

### Provision GitHub Actions WebHook to Trigger Tetkton CI Pipeline w/ Tekton Triggers



## Installation w/ ArgoCD

## Installation w/ FluxCD
