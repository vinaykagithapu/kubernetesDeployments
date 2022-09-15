# Helm - The package manager for Kubernetes
1. Helm is the best way to find, share, and use software built for Kubernetes.
2. Helm helps you manage Kubernetes applications. Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.
3. Charts are easy to create, version, share, and publish.

# Getting Started
## Install Helm
1. You can download specific version from [here](https://github.com/helm/helm/releases)
```shell
wget https://get.helm.sh/helm-v3.9.4-linux-amd64.tar.gz 
tar -zxvf helm-v3.9.4-linux-amd64.tar.gz
```
2. Install helm
```shell
mv linux-amd64/helm ~/.local/bin/helm
echo "export PATH=$PATH:$HOME/.local/bin" >> ~/.bashrc
bash
```

# CleanUp
1. Remove helm
```shell
rm -rf linux-amd64
rm helm-v3.9.4-linux-amd64.tar.gz
rm ~/.local/bin/helm
```