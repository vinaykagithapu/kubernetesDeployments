# Jenkins Deployment - Helm
Jenkins is an open source automation server. It helps automate the parts of software development related to building, testing, and deploying, facilitating continuous integration and continuous delivery. It is a server-based system that runs in servlet containers such as Apache Tomcat.

## Requirements 
1. Kubernetes ([Kubernetes Installation](https://github.com/vinaykagithapu/dockerDepolyments/tree/main/k8s-cluster-kind))
2. Helm ([Helm Installation](../helm/README.md#install-helm))

# Getting Started
## Deploy Jenkins Helm Chart from Repo
1. Add jenkins helm repo to local.
```shell
helm repo add jenkins https://charts.jenkins.io
helm repo update
```
2. Install jenkins helm charts.
```shell 
helm upgrade --install jenkins-app jenkins/jenkins \
             --create-namespace \
             --namespace jenkins-ns \
             --version=4.2.2 \
             -f jenkins-values.yaml
```
3. Wait for 5-10 mins and proceed.

## Access Jenkins
1. Get your 'admin' user password by running
```shell
kubectl exec --namespace jenkins-ns -it svc/jenkins-app -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
```
2. Get the Jenkins URL to visit by running these commands in the same shell:
```shell
echo http://127.0.0.1:8080
kubectl --namespace jenkins-ns port-forward svc/jenkins-app 8080:8080
```
3. Access the Jenkins UI at http://127.0.0.1:8080
4. Configure security realm and authorization strategy

## Post Configuration
### Add Admin User 
1. Goto **Dashboard > Manage Jenkins > Configure Global Security**
2. Security Realm : **Jenkins own user database**
3. Authorization  : **Logged-in users can do anything**
4. **Disable** Allow anonymous read access
5. **Apply > Save**
6. It will redirect to login page
7. Visit homepage, it will redirect to **firstUser** creation page

### Create First Admin User
1. Provide details as follows(Note: please change the creds as per your requirement)
```console
Username         : admin
Password         : admin
Confirm password : admin
Full name        : Administrator
E-mail address   : admin@test.com
```

# CleanUp
1. Uninstall jenkins deployment
```shell
helm uninstall jenkins-app -n jenkins-ns
kubectl delete namespace jenkins-ns
```