# d3v0ps-ch4ll3ng3
## 1. Prerequisites
Install kubectl
```
sudo apt update & sudo apt upgrade
sudo apt install conntrack socat
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```
Install Minikube 
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube\nsudo mv minikube /usr/local/bin/
minikube start --driver=docker
minikube status
kubectl cluster-info
kubectl get all -A
```
Install helm and plugins
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
wget https://github.com/helmfile/helmfile/releases/download/v0.153.1/helmfile_0.153.1_linux_amd64.tar.gz
tar xvf helmfile_0.153.1_linux_amd64.tar.gz
sudo install helmfile /usr/bin
helm plugin install https://github.com/jkroepke/helm-secrets --version v4.4.2
helm plugin install https://github.com/databus23/helm-diff
```
Install GNUpg
```
sudo apt install gnupg
mkdir ~/.gnupg/
vim ~/.gnupg/gpg.conf
gpg --full-generate-key
gpg --list-keys
gpg --list--secretskeys
```
Install Sops
```
wget https://github.com/mozilla/sops/releases/download/v3.7.3/sops_3.7.3_amd64.deb
sudo dpkg -i sops_3.7.3_amd64.deb
```
## 2. Development
Create Git repository
```
cd github
git clone git@github.com:devopsfreelance-pro/d3v0ps-ch4ll3ng3.git
cd d3v0ps-ch4ll3ng3
git branch 1st-approach
git checkout 1st-approach
```
Create and configure Chart
```
mkdir charts
cd charts
helm create nginx-demo
cd ..
vim charts/nginx-demo/values.yaml
vim charts/nginx-demo/templates/deployment.yaml
vim charts/nginx-demo/templates/secret.yaml
```
Create and encrypt secrets
```
vim secrets.dev.yaml
vim secrets.stage.yaml
sops --encrypt --pgp 68AFCD306987C84BD62148D2A7F4F618DC55EA8D secrets.dev.yaml > secrets.dev.enc.yaml
sops --encrypt --pgp 68AFCD306987C84BD62148D2A7F4F618DC55EA8D  secrets.stage.yaml > secrets.stage.enc.yaml
rm secrets.dev.yaml
rm secrets.stage.yaml
```
Create helmfile and  custom values
```
vim helmfile.yaml
vim values.stage.yaml
vim values.dev.yaml
```
## 3. Deployment
Deploy charts to environments and test result
```
kubectl cluster-info
kubectl create namespace dev
kubectl create namespace stage
helmfile -e dev apply
helm ls -n dev
kubectl -n dev get all
kubectl port-forward -n dev svc/nginx-demo-dev 8080:80
helmfile -e stage apply
helm ls -n stage
kubectl -n stage get all 
kubectl port-forward -n stage svc/nginx-demo-stage 8080:80
```
