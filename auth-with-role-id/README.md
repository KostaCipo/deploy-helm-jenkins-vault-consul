Deploying Jenkins with Vault to Kubernetes running locally using minikube.

# Runbook

## Minikube
### Installation

<minikube installation>

Finally, start minikube
```bash
minikube start --vm-driver=none
```

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
#### Vault
[Full installation guide](https://learn.hashicorp.com/vault/getting-started-k8s/minikube)
##### To install from custom repo
Clone chart sources
```bash
git clone https://github.com/hashicorp/vault-helm.git
```
To install Vault
```bash
helm install --name vault --values values/vault.yaml vault-helm
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

## Kubernetes-Vault
Follow the guide described [here](https://github.com/Boostport/kubernetes-vault/blob/master/deployments/quick-start/README.md)

During installation you only need to process steps from 1.4 to 3.3 (not including the last command which deploys sample application!)

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