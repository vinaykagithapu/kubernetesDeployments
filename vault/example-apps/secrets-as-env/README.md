# Secret as Env using Annotations

## Requirements
1. K8s cluster, consul and vault. [click here](../../helm/README.md) to setup.

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
1. Create a policy to map service account to secrets..
```shell
kubectl -n vault exec -it vault-0 -- sh 
vault login
# Login with root taken

cat <<EOF > /home/vault/app-policy.hcl
path "secret/basic-secret/*" {
  capabilities = ["read"]
}
EOF
vault policy write basic-secret-policy /home/vault/app-policy.hcl
```
2. Now lets create a role that maps Kubernetes service account, used by pod, to a policy.
```shell
vault write auth/kubernetes/role/basic-secret-role \
   bound_service_account_names=basic-secret \
   bound_service_account_namespaces=example-app \
   policies=basic-secret-policy \
   ttl=1h
```
4. Now service account for pod can access all secrets under `secret/basic-secret/*`

## Create Secrets
1. Lets create some secrets.
```shell
vault secrets enable -path=secret/ kv
vault kv put secret/basic-secret/helloworld username=dbuser password=P@ssw0rd1!
exit
```

## Testing
1. Clone the repo
```shell
git clone https://github.com/vinaykagithapu/kubernetesDeployments.git
cd kubernetesDeployments/vault/example-apps/secrets-as-env/
```
2. Lets deploy app and see if it works:
```shell
kubectl create ns example-app
kubectl -n example-app apply -f deployment.yaml
kubectl -n example-app get pods
```
3. Once the pod is ready, the secret is injected into the pod at the following location as environment variables:
```shell
kubectl -n example-app get pod
kubectl -n example-app -it exec <pod-name> -- sh
cat /vault/secrets/helloworld
source /vault/secrets/helloworld
echo $USERNAME $PASSWORD
exit
```