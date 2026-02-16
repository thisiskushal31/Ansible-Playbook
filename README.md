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
└── README.md
```

## Usage

- **MongoDB**: See [mongodb/README.md](mongodb/README.md). Run the playbook for your cloud (e.g. `mongodb/gcp/main.yml`) with inventory and cloud-specific variables supplied by your orchestrator.
- To add more custom playbooks later (e.g. Redis, app bootstrap), add a top-level folder and follow a similar layout if needed.
