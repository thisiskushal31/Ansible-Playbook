# MongoDB 8 on Google Cloud (GCE)

Sets up a **MongoDB 8** replica set on Ubuntu instances in GCE. The replica set keyfile is retrieved from **Google Cloud Secret Manager** when the playbook runs (nothing sensitive is stored in the repo). Designed to be run from the **infra-orchestrator** (inventory and cloud-specific vars supplied there).

## Requirements

- GCE instances already created (e.g. via Terraform or your orchestration), with inventory groups `primary` and `secondary` defined.
- An SSH user (default `ubuntu`) that Ansible can use with sudo.
- The keyfile stored as a secret in Secret Manager; the account running the playbook (e.g. Jenkins or the Ansible controller) must have permission to read it (e.g. `roles/secretmanager.secretAccessor`).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `gcp_project` | *(required)* | **Cloud-specific.** GCP project ID containing the keyfile secret. Infra-orchestrator should pass this (e.g. from config or Terraform). |
| `mongo_key_auth_secret` | *(required)* | **Cloud-specific.** Secret Manager secret name that holds the keyfile value. Infra-orchestrator should pass this (e.g. from config or secret manager). |
| `mongo_version` | `8.0` | MongoDB version; this playbook supports **8.0 only**. |
| `mongo_repl_set` | `rs0` | Replica set name. |
| `mongo_port` | `27017` | Port mongod listens on. |
| `remote_user` | `ubuntu` | SSH user on the instances. |

MongoDB admin and app user credentials can be supplied via host/group vars (e.g. from Ansible Vault or another secret source). The `mongodb_keyfile_gcp` role only reads the keyfile from Secret Manager.

## Usage

When run from infra-orchestrator, inventory and the cloud-specific vars above are provided by the orchestrator. Standalone example:

```bash
cd mongodb/gcp
ansible-playbook -i hosts.ini main.yml -e "gcp_project=YOUR_PROJECT" -e "mongo_key_auth_secret=YOUR_KEYFILE_SECRET"
```

## Inventory

Copy `hosts.ini.example` to `hosts.ini` and set the primary and secondary hostnames or IPs. When used from infra-orchestrator, inventory is typically generated from Terraform outputs or a dynamic inventory plugin.
