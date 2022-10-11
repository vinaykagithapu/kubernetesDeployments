# Certmanager - Letsencrypt - Kind
1. cert-manager adds certificates and certificate issuers as resource types in Kubernetes clusters, and simplifies the process of obtaining, renewing and using those certificates.
2. It can issue certificates from a variety of supported sources, including Let's Encrypt, HashiCorp Vault, and Venafi as well as private PKI.
3. It will ensure certificates are valid and up to date, and attempt to renew certificates at a configured time before expiry.

## Requirements
1. [Click here](https://github.com/vinaykagithapu/kubernetesDeployments/blob/main/metalLB/README.md) to setup k8s cluster. 

# Getting started
## Deploy Nginx Ingress Controller
1. Nginx ingress controller is deployed with the helm chart
```shell
helm upgrade --install ingress-nginx \
    ingress-nginx --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx \
    --create-namespace
helm list -A
kubectl get all -n ingress-nginx
```
2. wait for 2 mins

## Deploy CertManager
1. CertManager is deployed with the cert-manager manifest
```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
kubectl get all -n cert-manager
```
2. wait of 2 mins

## Create ClusterIssuer
```shell
kubectl get crds | grep -i cert-manager
kubectl apply -f clusterissuer.yaml
```

## Sample Nginx Deployment
```shell
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port 80
kubectl get all
```

## Create Ingress
```shell
kubectl apply -f ingress.yaml 
kubectl get ingress
kubectl describe ingress ingress-resource
```

## Testing
```shell
export LB_IP=$(kubectl get ingress -o jsonpath="{.items[0].status.loadBalancer.ingress[0].ip}")
echo "$LB_IP vinay.example.com" | sudo tee -a /etc/hosts
```
```shell
curl vinay.example.com
curl -L vinay.example.com
curl -Lk vinay.example.com
curl -Lkv vinay.example.com
```

# CleanUp
```shell
kubectl delete -f ingress.yaml
kubectl delete service nginx
kubectl delete deployments nginx
kubectl delete -f clusterissuer.yaml
kubectl delete -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
helm unistall ingress-nginx -n ingress-nginx
```