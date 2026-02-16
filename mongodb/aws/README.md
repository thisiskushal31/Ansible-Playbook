# MongoDB on AWS (EC2)

Sets up a MongoDB replica set on Ubuntu EC2 instances. The replica set keyfile is loaded from **AWS Secrets Manager** at runtime.

## Requirements

- EC2 instances already created (e.g. via Terraform or your orchestration), with inventory groups `primary` and `secondary`.
- IAM permissions so the context running the playbook (instance profile or Jenkins/Ansible credentials) can call `secretsmanager:GetSecretValue` for the keyfile secret.
- AWS CLI available where the secret is fetched (target instance or control node), with credentials configured.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `mongo_key_auth_secret` | *(required)* | **Cloud-specific.** Secrets Manager secret name or ARN for the keyfile. Infra-orchestrator should pass this. |
| `aws_region` | `us-east-1` | **Cloud-specific.** Region where the secret is stored. Infra-orchestrator should pass this. |
| `mongo_version` | `8.0` | This playbook installs **MongoDB 8 only**. |
| `mongo_repl_set` | `rs0` | Replica set name. |
| `mongo_port` | `27017` | Port mongod listens on. |
| `remote_user` | `ubuntu` | SSH user on the instances. |

## Usage

```bash
cd mongodb/aws
ansible-playbook -i hosts.ini main.yml -e "mongo_key_auth_secret=prod/mongodb/keyfile" -e "aws_region=us-east-1"

ansible-playbook -i hosts.ini main.yml -e "mongo_key_auth_secret=prod/mongodb/keyfile" -e "aws_region=us-east-1"
```

The secret in Secrets Manager should contain the raw keyfile string (same format as used on GCP). Ensure the IAM role or user used to run the playbook has `secretsmanager:GetSecretValue` on this secret.
