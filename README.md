Deploying Jenkins with Vault to Kubernetes running locally using minikube.

# Runbook

## Minikube
### Installation
Install kvm and dependencies
Validate that libvirt reports no errors
```bash
virt-host-validate
```
If you see this line
> QEMU: Checking if device /dev/kvm is accessible: ***FAIL*** (Check /dev/kvm is world writable or you are in a group that is allowed to access it)
 You need to add your user to 'kvm' group and logout/restart:
```bash
sudo adduser $USER kvm
```
[How to check user relates to kvm group](https://stackoverflow.com/questions/37300811/android-studio-dev-kvm-device-permission-denied)<br>

<minikube installation>

Finally, start minikube
```bash
minikube start --vm-driver kvm2
```
To make kvm2 the default driver and start minikube
```bash
minikube config set vm-driver kvm2
minikube start
```
[How to set kvm2 as default driver](https://minikube.sigs.k8s.io/docs/reference/drivers/kvm2/)<br>
### Useful possibilities
To check minikube cluster status
```bash
minikube status
```
To show minikube dashboard in your browser
```bash
minikube dashboard
```
## Helm
### Installation
To install
```bash
sudo snap install helm --classic
```
To install Tiller into existent cluster
```bash
helm init
```
### Working with Helm
To install Helm chart
```bash
helm install --name <NAME> --values <VALUES_FILE> <REPO_OR_PATH>
```
To delete Helm package
```bash
helm delete <NAME>
```
or
```bash
helm del --purge <NAME>
```

## Vault (with Consul)
### Installation
#### Consul
Create values.yaml file to bind needed values
```
# Choose an optional name for the datacenter
global:
  datacenter: dc

# Enable the Consul Web UI via a NodePort
ui:
  service:
    type: 'NodePort'

# Enable Connect for secure communication between nodes
connectInject:
  enabled: true

client:
  enabled: true
  grpc: true

# Use only one Consul server for local development
server:
  replicas: 1
  bootstrapExpect: 1
  disruptionBudget:
    enabled: true
    maxUnavailable: 0
```
Deploy Consul chart using Helm
```bash
helm install --values values.yaml --set Replicas=1 --name consul stable/consul
```
Check newly created pods
```bash
kubectl get pods
```
To explore Consul web UI
```bash
minikube service list
minikube service consul-ui
```
#### Vault
[Full installation guide](https://learn.hashicorp.com/vault/getting-started-k8s/minikube)
##### To install from Helm repo:
Add 'incubator' repo to Helm
```bash
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
```
To install Vault
```bash
helm install --name vault --set 'authDelegator.enabled=true' --set vault.config.storage.consul.address="consul:8500" --set replicaCount=1,tlsDisable=true incubator/vault
```
##### To install from custom repo
Clone chart sources
```bash
git clone https://github.com/hashicorp/vault-helm.git
```
To install Vault
```bash
helm install --name vault --values values/vault.yml --set 'authDelegator.enabled=true' vault-helm
```
##### Next
To install Vault locally
```bash
sudo snap install vault
```
To prevent any issues with connection you need to export Vault address as envvar
```bash
export VAULT_ADDR='http://127.0.0.1:8200'
```
To get vault status
```bash
kubectl exec vault-0 -- vault status
```
Get seal key and root token
```bash
kubectl exec vault-0 -- vault operator init -key-shares 1 -key-threshold 1
```
To unseal Vault
```bash
kubectl exec vault-0 -- vault operator unseal <UNSEAL_KEY>
```
Set port forwarding
```bash
kubectl port-forward vault-0 8200:8200
```
Login using root token
```bash
vault login <ROOT_TOKEN>
```
##### Set secret in Vault
Enable kv-v2 secrets
```bash
vault secrets enable -path=secret kv-v2
```
Put secret
```bash
vault kv put <path/to/secret> username="<USERNAME>" password="<PASSWORD>"
```
Get newly created secret from Vault
```bash
vault kv get <path/to/secret>
``` 
**Path: secret/jenkins/git**

## Jenkins
```bash
helm install --name jenkins --values values/jenkins.yml stable/jenkins
```

## Starting Jenkins
```bash
minikube service jenkins
```
Default login: admin
Default password: admin

BE AWARE OF USING THESE CREDENTIALS IN PRODUCTION MODE! Only for testing purposes.