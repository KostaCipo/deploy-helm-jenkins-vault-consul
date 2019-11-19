This project shows different approaches to integrate Jenkins with Vault using Kubernetes environment.

# auth-with-root-token
The most common way to use Vault-stored secrets in Jenkins using authentication by Vault root token which generated on early stages of Vault setup.

# auth-with-role-id
This way just like previous but a much more securely - AppRole uses instead of root tokens.

# Installation guide
```bash
git clone https://github.com/WD250RXS98UH42JS/deploy-helm-jenkins-vault-consul.git
```

Each part contains own runbook itself. Just read README.md and follow step-by-step recommendations. 