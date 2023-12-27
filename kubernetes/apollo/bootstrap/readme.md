# Bootstrap Flux

## 1. Install the Flux manifests into the cluster

```sh
  kubectl apply --kustomize ./kubernetes/apollo/bootstrap/flux
```

## 2. Apply Cluster Secrets and ConfigMaps needed before bootstrapping this Git Repository

_These cannot be applied with `kubectl` in the regular fashion due to be encrypted with sops_

```sh
sops --decrypt kubernetes/apollo/bootstrap/flux/age-key.sops.yaml | kubectl apply -f -
sops --decrypt kubernetes/apollo/bootstrap/flux/github-deploy-key.sops.yaml | kubectl apply -f -
sops --decrypt kubernetes/apollo/flux/vars/cluster-secrets.sops.yaml | kubectl apply -f -
sops --decrypt kubernetes/apollo/flux/vars/aws-secret.yaml | kubectl apply -f -
kubectl apply -f kubernetes/apollo/flux/vars/cluster-settings.yaml
```

## 3. Apply the Flux CRs to bootstrap this Git repository into the cluster

```sh
kubectl apply --kustomize ./kubernetes/apollo/flux/config
```