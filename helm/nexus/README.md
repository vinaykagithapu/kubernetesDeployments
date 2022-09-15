# Nexus Deployment - Helm
Nexus is a repository manager, it stores "artifacts". It allows you to proxy, collect, and manage your dependencies so that you are not constantly juggling a collection of JARs. It makes it easy to distribute your software. Internally, you configure your build to publish artifacts to Nexus and they then become available to other developers. You get the benefits of having your own 'central'. 

## Requirements 
1. Kubernetes ([Kubernetes Installation](https://github.com/vinaykagithapu/dockerDepolyments/tree/main/k8s-cluster-kind))
2. Helm ([Helm Installation](../README.md#install-helm))

# Getting Started
## Deploy Nexus Helm Chart from Repo
1. Add nexus helm repo to local.
```shell
helm repo add sonatype https://sonatype.github.io/helm3-charts/
helm repo update
```
2. Modify the [nexus-values.yaml](./nexus-values.yaml) based on requirement.
3. Install nexus helm charts.
```shell 
helm upgrade --install nexus-rm-app sonatype/nexus-repository-manager \
             --create-namespace \
             --namespace nexus-ns \
             --version=41.1.3 \
             -f nexus-values.yaml
```
4. Wait for 4-5 mins and proceed.

## Access nexus
1. Get your 'admin' user password by running
```shell
export POD_NAME=$(kubectl get pods --namespace nexus-ns -l "app.kubernetes.io/name=nexus-repository-manager,app.kubernetes.io/instance=nexus-rm-app" -o jsonpath="{.items[0].metadata.name}")
kubectl exec --namespace nexus-ns -it $POD_NAME -- /bin/cat /nexus-data/admin.password && echo
```
2. Get the nexus URL to visit by running these commands in the same shell:
```shell
echo http://127.0.0.1:8081
kubectl --namespace nexus-ns port-forward svc/nexus-rm-app-nexus-repository-manager 8081:8081
```
3. Access the nexus UI at http://127.0.0.1:8081. 
4. Login with the password from step 1 and the username: admin

## Post Configuration
### Setup New Password 
1. Click on **Next > New password: > Confirm password: > Next**
2. **Disable** Disable anonymous access > **Next > Finish**

# CleanUp
1. Uninstall nexus deployment
```shell
helm uninstall nexus-rm-app -n nexus-ns
kubectl delete namespace nexus-ns
```