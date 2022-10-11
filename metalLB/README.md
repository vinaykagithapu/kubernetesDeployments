# MetalLB - Kind Cluster
MetalLB hooks into your Kubernetes cluster, and provides a network load-balancer implementation. In short, it allows you to create Kubernetes services of type LoadBalancer in clusters that don’t run on a cloud provider, and thus cannot simply hook into paid products to provide load balancers. 

# Getting Started
1. Clone the repo
```shell
git clone https://github.com/vinaykagithapu/kubernetesDeployments.git
cd kubernetesDeployments/metalLB
```
## Create k8s cluster
1. Kind is used to create k8s cluster
```shell
kind create cluster --name metalLBCluster
``` 
## Deploy MetalLB
1. To deploy metalLB, apply the below manifest
```shell
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.5/config/manifests/metallb-native.yaml
kubectl get all -n metallb-system
```

## Configuration
1. To deploy an IPaddress pool, we need IP ranges as node IPs
```shell
kubectl get node -owide
```
2. Modify the IP address range in [ipaddresspool.yaml](./ipaddresspool.yaml)
3. Create IPAddressPool resource
```shell
kubectl apply -f ipaddresspool.yaml
```
4. Modify the IPAddress pool in [l2advertisement.yaml](./l2advertisement.yaml)
5. Create L2Advertisement resource
```shell
kubectl apply -f l2advertisement.yaml
```

## Testing
1. Create a sample nginx deployment
```shell
kubectl create deployment nginx --image=nginx
```
2. Expose nginx deployment as LoadBalancer
```shell
kubectl expose deployment nginx --port 80 --type=LoadBalancer 
kubectl get svc
```
3. Access the nginx service via LoadBalancer
```shell
export LB_IP=$(kubectl get service nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo http://$LB_IP
```

# CleanUp
```shell
kubectl delete service nginx
kubectl delete deployments nginx
kubectl delete -f l2advertisement.yaml
kubectl delete -f ipaddresspool.yaml
kubectl delete -f https://raw.githubusercontent.com/metallb/metallb/v0.13.5/config/manifests/metallb-native.yaml
kind delete cluster --name metalLBCluster
```