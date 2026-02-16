# MongoDB on VMs (Ansible)

Playbooks and roles to install and configure MongoDB replica sets on cloud VMs. Use these **after** your orchestration layer has already provisioned the instances (GCE, EC2, or Azure).

## Typical workflow

1. **Provision** – Your automation (e.g. Terraform, scripts, or Jinja-based templates) creates one or more VMs and assigns them roles (e.g. primary vs secondary).
2. **Configure** – A CI/CD job (e.g. Jenkins) runs the matching Ansible playbook from this tree, targeting those hosts.
3. **Secrets** – Credentials and the replica set keyfile are **not** in this repo. They are loaded at runtime from the provider’s secret store (GCP Secret Manager, AWS Secrets Manager, or Azure Key Vault), with access controlled via IAM in the environment where the playbook runs.

## Directory layout

```
mongodb/
├── README.md                 # This file
├── common/
│   └── roles/                # Shared roles (install, replica set, users, node_exporter)
│       ├── mongodb_install          # Install MongoDB 8, repo, packages, limits, data dir
│       ├── mongodb_apt_repo        # Configure MongoDB APT repository
│       ├── mongodb_replica_bootstrap  # Initiate replica set on primary
│       ├── mongodb_admin_users     # Create admin users (primary)
│       ├── mongodb_app_users       # Create application users
│       ├── mongodb_replica_join    # Add secondary nodes to replica set
│       ├── mongodb_wait_primary    # Wait for primary election
│       ├── mongodb_wait_listening  # Wait for mongod port
│       ├── mongodb_restart         # Restart mongod service
│       └── node_exporter           # Prometheus node_exporter for metrics
├── gcp/                      # Google Cloud (GCE) – keyfile from Secret Manager
│   ├── README.md
│   ├── main.yml
│   ├── ansible.cfg
│   ├── hosts.ini.example
│   └── roles/
│       └── mongodb_keyfile_gcp
├── aws/                      # Amazon Web Services (EC2) – keyfile from Secrets Manager
│   ├── README.md
│   ├── main.yml
│   ├── ansible.cfg
│   ├── hosts.ini.example
│   └── roles/
│       └── mongodb_keyfile_aws
└── azure/                    # Microsoft Azure VMs – keyfile from Key Vault
    ├── README.md
    ├── main.yml
    ├── ansible.cfg
    ├── hosts.ini.example
    └── roles/
        └── mongodb_keyfile_azure
```

## Running the playbooks

- **Google Cloud** – Use an inventory that includes your GCE hosts (e.g. from Terraform or dynamic inventory), then:
  ```bash
  cd mongodb/gcp && ansible-playbook -i hosts.ini main.yml -e "gcp_project=YOUR_PROJECT" -e "mongo_key_auth_secret=YOUR_SECRET_NAME"
  ```
- **AWS** – Run `mongodb/aws/main.yml` with an EC2 inventory and the required extra vars (see `mongodb/aws/README.md`).
- **Azure** – Run `mongodb/azure/main.yml` with an Azure VM inventory and the required extra vars (see `mongodb/azure/README.md`).

Inventory can be static (e.g. a generated `hosts.ini`) or dynamic; your pipeline can produce it from the orchestration layer’s output.

## Configuration and secrets

- **MongoDB version**: These playbooks install **MongoDB 8.0 only** (`mongo_version` default `8.0`).
- **Replica set**: `mongo_repl_set` (default `rs0`).
- **Keyfile**: Each cloud subdirectory uses a role that reads the replica set keyfile from that provider’s secret store and writes it on the targets. The run environment (VMs or control node) must have the right IAM/permissions to read that secret.

Exact variable names and examples are in each cloud’s `README.md`.
