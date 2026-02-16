# MongoDB on Azure (VMs)

Sets up a MongoDB replica set on Ubuntu Azure VMs. The replica set keyfile is loaded from **Azure Key Vault** at runtime.

## Requirements

- Azure VMs already created (e.g. via Terraform or your orchestration), with inventory groups `primary` and `secondary`.
- The keyfile stored as a secret in Key Vault. The identity used to run the playbook (VM managed identity or Ansible control node) must have **Get** on that secret (e.g. Key Vault Secrets User).
- Azure CLI (`az`) installed and authenticated on the host that fetches the secret (target VM with managed identity or control node).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `azure_keyvault_name` | *(required)* | **Cloud-specific.** Key Vault name. Infra-orchestrator should pass this. |
| `mongo_key_auth_secret` | *(required)* | **Cloud-specific.** Name of the secret in Key Vault that holds the keyfile. Infra-orchestrator should pass this. |
| `mongo_version` | `8.0` | This playbook installs **MongoDB 8 only**. |
| `mongo_repl_set` | `rs0` | Replica set name. |
| `mongo_port` | `27017` | Port mongod listens on. |
| `remote_user` | `ubuntu` | SSH user on the VMs. |

## Usage

```bash
cd mongodb/azure
az login
ansible-playbook -i hosts.ini main.yml -e "azure_keyvault_name=my-keyvault" -e "mongo_key_auth_secret=mongodb-keyfile"

ansible-playbook -i hosts.ini main.yml -e "azure_keyvault_name=my-keyvault" -e "mongo_key_auth_secret=mongodb-keyfile"
```

Create the secret in Key Vault (e.g. Azure Portal or `az keyvault secret set`). Grant the VM’s managed identity or the identity used by Ansible at least **Key Vault Secrets User** (or Get on that secret).
