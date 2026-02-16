# Ansible Role: Node Exporter

[![CI](https://github.com/geerlingguy/ansible-role-node_exporter/workflows/CI/badge.svg?event=push)](https://github.com/geerlingguy/ansible-role-node_exporter/actions?query=workflow%3ACI)

Installs and configures [Prometheus Node Exporter](https://github.com/prometheus/node_exporter) on Linux systems.

## Requirements

- Ansible 2.9+
- Target system must be able to reach GitHub API (for latest version detection)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `node_exporter_version` | `'latest'` | Node Exporter version to install |
| `node_exporter_skip_github_api` | `false` | Skip GitHub API call for latest version |
| `node_exporter_arch` | `'amd64'` | Architecture (amd64, arm64, etc.) |
| `node_exporter_bin_path` | `/usr/local/bin/node_exporter` | Binary installation path |
| `node_exporter_port` | `9100` | Port to expose metrics on |
| `node_exporter_state` | `started` | Service state |
| `node_exporter_enabled` | `true` | Enable service at boot |

## Usage

### Basic Installation

```yaml
- hosts: monitoring_servers
  roles:
    - node_exporter
```

### Skip GitHub API (Offline/Blocked Environments)

If you're in an environment where GitHub API is blocked or you want to avoid the API call:

```yaml
- hosts: monitoring_servers
  vars:
    node_exporter_skip_github_api: true
    node_exporter_version: '1.6.1'  # Use specific version
  roles:
    - node_exporter
```

### Use Specific Version

```yaml
- hosts: monitoring_servers
  vars:
    node_exporter_version: '1.6.1'
  roles:
    - node_exporter
```

## Troubleshooting

### GitHub API Issues

If you encounter errors like:
```
fatal: [host]: FAILED! => {"msg": "The conditional check '_github_release.status == 200' failed. The error was: error while evaluating conditional (_github_release.status == 200): 'dict object' has no attribute 'status'"}
```

**Solutions:**

1. **Skip GitHub API call:**
   ```yaml
   node_exporter_skip_github_api: true
   node_exporter_version: '1.6.1'
   ```

2. **Check network connectivity:**
   - Ensure the Ansible controller can reach `api.github.com`
   - Check for firewall/proxy restrictions

3. **Use specific version:**
   ```yaml
   node_exporter_version: '1.6.1'
   ```

### Common Issues

1. **Permission denied**: Ensure the target user has sudo privileges
2. **Port already in use**: Change `node_exporter_port` if 9100 is occupied
3. **Binary not found**: Check `node_exporter_bin_path` configuration

## Dependencies

None.

## License

MIT / BSD
