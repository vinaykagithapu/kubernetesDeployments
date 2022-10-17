# Mount secrets using CSI driver
## Requirements
1. K8s cluster, consul, vault and secrets-store-csi-driver. [click here](../../vault-csi/README.md) to setup.

## Enable Kubernetes Authentication
1. For the injector to be authorised to access vault, we need to enable K8s auth
```shell
kubectl -n vault exec -it vault-0 -- sh 

vault login
# Login with root taken
vault auth enable kubernetes

vault write auth/kubernetes/config \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
issuer="https://kubernetes.default.svc.cluster.local"
exit
```

## Setup a policy.
1. create the policy to map service account to secrets.
```shell
kubectl -n vault exec -it vault-0 -- sh 
vault login
# Login with root taken

cat <<EOF > /home/vault/app-policy.hcl
path "secret/db-pass" {
  capabilities = ["read"]
}
EOF
vault policy write internal-app /home/vault/app-policy.hcl
```

2. Now lets create a role that maps Kubernetes service account, used by pod, to a policy.
```shell
vault write auth/kubernetes/role/database \
    bound_service_account_names=webapp-sa \
    bound_service_account_namespaces=example-app-ns \
    policies=internal-app \
    ttl=20m
```
4. Service account for pod can access all secrets under `secret/db-pass`

## Create Secrets
1. Lets create some secrets.
```shell
vault secrets enable -path=secret/ kv
vault kv put secret/db-pass password="db-secret-P@ssw0rd1!"
vault kv get secret/db-pass
exit
```

## Testing
1. Clone the repo
```shell
git clone https://github.com/vinaykagithapu/kubernetesDeployments.git
cd kubernetesDeployments/vault/example-apps/csi-volume-secret/
```
2. Lets deploy app and see if it works:
```shell
kubectl create ns example-app-ns
kubectl -n example-app-ns apply -f spc-vault-database.yaml
kubectl -n example-app-ns describe SecretProviderClass vault-database
kubectl -n example-app-ns create serviceaccount webapp-sa
kubectl -n example-app-ns apply -f webapp-pod.yaml 
kubectl -n example-app-ns get pods
```
3. Once the pod is ready, the secret volume is mounted to the pod at the following location:
```shell
kubectl -n example-app-ns get pod
kubectl -n example-app-ns exec webapp -- sh -c "cat /mnt/secrets-store/db-password"
```