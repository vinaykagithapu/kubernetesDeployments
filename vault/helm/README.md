# Vault - Kind Cluster
1. Vault secures, stores, and tightly controls access to tokens, passwords, certificates, API keys, and other secrets in modern computing.
2. Vault comes with various pluggable components called secrets engines and authentication methods allowing you to integrate with external systems.
3. The purpose of those components is to manage and protect your secrets in dynamic infrastructure (e.g. database credentials, passwords, API keys).

## Requirements
### Create k8s cluster
1. Kind is used to create k8s cluster
```shell
kind create cluster --name vault-cluster
```

# Getting Started
1. Clone the repo
```shell
git clone https://github.com/vinaykagithapu/kubernetesDeployments.git
cd kubernetesDeployments/vault/helm
```

## Install Vault
1. Deploy consul and vault using helm charts.
```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm upgrade --install consul hashicorp/consul \
            --namespace vault --create-namespace \
            --version 0.49.0 \
            -f consul-values.yaml \
            --wait
helm upgrade --install vault hashicorp/vault \
            --namespace vault --create-namespace \
            --version 0.22.0 \
            -f vault-values.yaml \
            --wait
```

## Configure Vault
1. Exec into vault pod, initialize the vault and copy the root token and unseal keys.
```shell
kubectl -n vault exec -it vault-0 -- sh
vault operator init 
```
2. Unseal the vault with any 3 unseal keys
```shell
kubectl -n vault exec -it vault-0 -- sh
vault operator unseal
# Paste unseal key 1
vault operator unseal
# Paste unseal key 2
vault operator unseal
# Paste unseal key 3 
exit
```

## Access Vault-UI
1. Port-forward the  vault-ui service
```shell
kubectl -n vault port-forward service/vault-ui 8200:8200
```
2. Acces UI at http://127.0.0.1:8200/ui/
3. Login with the root token copied earlier.

## Basic Secret Injection using Annotations
1. [Click here](../example-apps/basic-secret/README.md) to setup basic secret injection using vault annotation.

# CleanUp
```shell
helm uninstall vault -n vault
helm uninstall consul -n vault
```