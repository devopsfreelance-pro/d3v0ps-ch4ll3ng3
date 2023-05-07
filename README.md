# d3v0ps-ch4ll3ng3
## 1. Prerequisites
Install kubectl
```bash
sudo apt update & sudo apt upgrade
sudo apt install conntrack socat
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
```
Install Minikube 
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube\nsudo mv minikube /usr/local/bin/
minikube start --driver=docker
minikube status
kubectl cluster-info
kubectl get all -A
```
Install helm and plugins
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
wget https://github.com/helmfile/helmfile/releases/download/v0.153.1/helmfile_0.153.1_linux_amd64.tar.gz
tar xvf helmfile_0.153.1_linux_amd64.tar.gz
sudo install helmfile /usr/bin
helm plugin install https://github.com/jkroepke/helm-secrets --version v4.4.2
helm plugin install https://github.com/databus23/helm-diff
```
Install GNUpg
```bash
sudo apt install gnupg
mkdir ~/.gnupg/
vim ~/.gnupg/gpg.conf
gpg --full-generate-key
gpg --list-keys
gpg --list--secretskeys
```
Install Sops
```bash
wget https://github.com/mozilla/sops/releases/download/v3.7.3/sops_3.7.3_amd64.deb
sudo dpkg -i sops_3.7.3_amd64.deb
```
## 2. Development
Create Git repository
```bash
git clone git@github.com:devopsfreelance-pro/d3v0ps-ch4ll3ng3.git
cd d3v0ps-ch4ll3ng3
git branch 1st-approach
git checkout 1st-approach
```
Create and configure Chart
```bash
mkdir charts
cd charts
helm create nginx-demo
cd ..
```
Edit the following files:

vim [charts/nginx-demo/values.yaml](./charts/nginx-demo/values.yaml) \

```yaml
image:
  repository: nginxdemos/hello
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""
```
vim [charts/nginx-demo/templates/deployment.yaml](./charts/nginx-demo/templates/deployment.yaml) \

```yaml
          env:
            - name: ENVIRONMENT
              value: {{ .Values.environment }}
            - name: SECRET_VALUE
              valueFrom:
                secretKeyRef:
                  name: {{ .Release.Name }}-secret
                  key: secretValue

```
vim [charts/nginx-demo/templates/secret.yaml](./charts/nginx-demo/templates/secret.yaml) 

```yaml
ApiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
data:
  secretValue: {{ .Values.secretValue | b64enc | quote }}
```
Create and encrypt secrets 

vim secrets.dev.yaml

```yaml
secretValue: my-dev-secret
```
vim secrets.stage.yaml
```yaml
secretValue: my-stage-secret
```
Encrypt files 
```bash
sops --encrypt --pgp 68AFCD306987C84BD62148D2A7F4F618DC55EA8D secrets.dev.yaml > secrets.dev.enc.yaml
sops --encrypt --pgp 68AFCD306987C84BD62148D2A7F4F618DC55EA8D  secrets.stage.yaml > secrets.stage.enc.yaml
rm secrets.dev.yaml
rm secrets.stage.yaml
```
Create helmfile and  custom values \

vim [helmfile.yaml](./helmfile.yaml) \

```yaml
environments:
  dev:
    values:
      - environment: dev
  stage:
    values:
      - environment: stage

---
releases:
  - name: nginx-demo-{{ .Environment.Name }}
    namespace: {{ .Environment.Name }}
    chart: ./charts/nginx-demo
    values:
      - values.{{ .Environment.Name }}.yaml
    secrets:
      - secrets.{{ .Environment.Name }}.enc.yaml
```
vim [values.stage.yaml](./values.stage.yaml) \

```yaml
image:
  repository: nginxdemos/hello
  tag: plain-text
replicaCount: 3
```
vim [values.dev.yaml](./values.dev.yaml)

```yaml
image:
  repository: nginxdemos/hello
  tag: plain-text
replicaCount: 1
```
## 3. Deployment
Deploy charts to environments and test result
```bash
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
