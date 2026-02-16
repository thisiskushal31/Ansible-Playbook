# Ansible playbooks – custom configuration

This repository holds Ansible playbooks and roles for your custom setup. Run them from your orchestration layer (e.g. infra-orchestrator / Jenkins) after infrastructure is provisioned.

## Structure

```
Ansible-Playbook/
├── mongodb/        # MongoDB 8 replica set on VMs (GCP, AWS, Azure)
│   ├── common/     # Shared roles (install, replica set, users, node_exporter)
│   ├── gcp/        # GCE + GCP Secret Manager
│   ├── aws/        # EC2 + AWS Secrets Manager
│   └── azure/      # Azure VM + Key Vault
├── elasticsearch/  # Elasticsearch 9.x (same layout as mongodb: common + gcp/aws/azure)
│   ├── common/     # elasticsearch_install, elasticsearch_config
│   ├── gcp/        # GCE + GCP Secret Manager → keystore
│   ├── aws/        # EC2 + AWS Secrets Manager → keystore
│   └── azure/      # Azure VM + Key Vault → keystore
└── README.md
```

## Usage

- **MongoDB**: See [mongodb/README.md](mongodb/README.md). Run the playbook for your cloud (e.g. `mongodb/gcp/main.yml`) with inventory and cloud-specific variables supplied by your orchestrator.
- **Elasticsearch**: See [elasticsearch/README.md](elasticsearch/README.md). Run the playbook for your cloud (e.g. `elasticsearch/gcp/main.yml`). All group names, discovery, and counts come from inventory/vars; secrets from the cloud’s secret manager (keystore).
- To add more custom playbooks later (e.g. Redis, app bootstrap), add a top-level folder and follow a similar layout if needed.
