# Elasticsearch 9.x (MongoDB-style layout)

Production-grade Ansible for Elasticsearch 9.x. **No hardcoding**: group names, node counts, discovery seed hosts, and initial master nodes come from **inventory and variables** (hosts.ini or group_vars). Secrets are pulled from the cloud secret manager (GCP Secret Manager, AWS Secrets Manager, or Azure Key Vault) at runtime, same pattern as the MongoDB playbooks.

## Structure (same as MongoDB)

```
elasticsearch/
├── README.md
├── common/
│   └── roles/
│       ├── elasticsearch_install   # Repo, package, dirs, JVM, systemd
│       └── elasticsearch_config    # elasticsearch.yml from vars (node.roles, discovery)
├── gcp/
│   ├── main.yml
│   ├── ansible.cfg
│   ├── hosts.ini.example
│   └── roles/
│       └── elasticsearch_keystore_gcp   # Secret from GCP Secret Manager → ES keystore
├── aws/
│   ├── main.yml
│   ├── ansible.cfg
│   ├── hosts.ini.example
│   └── roles/
│       └── elasticsearch_keystore_aws
└── azure/
    ├── main.yml
    ├── ansible.cfg
    ├── hosts.ini.example
    └── roles/
        └── elasticsearch_keystore_azure
```

## Inventory format

You define **which hosts are master, data, coordinating, or ingest** by placing their IPs/hostnames under the right sections. The playbook uses **variable group names** (defaults: `node.master`, `node.data`, `node.coordinating`, `node.ingest`). Override them in `[all:vars]` if you use different section names.

- **Master**: cluster management only.
- **Data**: store indices (or use `es_data_master=1` for combined data+master).
- **Coordinating**: client nodes; route requests, no storage.
- **Ingest**: run ingest pipelines; no data storage.

Discovery seed hosts and initial master nodes are **not** hardcoded: set `es_discovery_seed_hosts` and `es_initial_master_nodes` in group_vars (or pass via `-e`). If you leave them unset, the playbook derives them from the group named by `es_group_master` (so the list of IPs and node names comes from the inventory).

## Variables (all from inventory / group_vars / -e)

| Variable | Purpose |
|----------|---------|
| `es_group_master`, `es_group_data`, `es_group_coordinating`, `es_group_ingest` | Inventory group names (defaults: node.master, node.data, …). |
| `es_discovery_seed_hosts` | List of master-eligible hostnames/IPs. Omit to use `groups[es_group_master]`. |
| `es_initial_master_nodes` | List of full node names for bootstrap. Omit to compute from `es_group_master`. |
| `es_version`, `es_cluster_name`, `es_node_name_prefix` | Cluster and node identity. |
| `es_master`, `es_data`, `es_data_master`, `es_coordinating`, `es_ingest` | Role flags per host (set in group_vars per group or per host). |
| Cloud secret vars | GCP: `gcp_project`, `es_keystore_secret_name`, `es_keystore_key_name`. AWS: `es_keystore_secret_name`, `aws_region`. Azure: `azure_keyvault_name`, `es_keystore_secret_name`. |

## Usage

- **GCP**: `cd elasticsearch/gcp && ansible-playbook -i hosts.ini main.yml` (set `gcp_project`, `es_keystore_secret_name`, etc. in inventory or -e).
- **AWS**: `cd elasticsearch/aws && ansible-playbook -i hosts.ini main.yml`.
- **Azure**: `cd elasticsearch/azure && ansible-playbook -i hosts.ini main.yml`.

Copy the relevant `hosts.ini.example` to `hosts.ini`, fill in IPs and vars. Your orchestrator can generate inventory from Terraform outputs.
