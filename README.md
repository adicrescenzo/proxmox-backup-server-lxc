# Proxmox Backup Server LXC Collection

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-adicrescenzo.proxmox__backup__server__lxc-blue.svg)](https://galaxy.ansible.com/adicrescenzo/proxmox_backup_server_lxc)

An Ansible collection for automating the deployment and configuration of Proxmox Backup Server (PBS) in LXC containers on Proxmox VE. This collection provides a complete solution for creating, configuring, and managing PBS instances with support for NFS storage, TLS certificates, and both Community and Enterprise editions.

## Features

- 🚀 **Automated LXC Container Creation** - Create and configure LXC containers on Proxmox VE
- 🔧 **Complete PBS Installation** - Install and configure Proxmox Backup Server (Community & Enterprise)
- 💾 **NFS Storage Support** - Automatic NFS mount configuration for backup storage
- 🔒 **TLS Certificate Management** - Deploy custom TLS certificates for secure connections
- 🏗️ **Infrastructure as Code** - Fully declarative configuration management
- 🔄 **Lifecycle Management** - Create, configure, and delete PBS instances

## Requirements

- **Ansible**: 2.1 or later
- **Proxmox VE**: 7.0 or later with API access
- **Target OS**: Debian 11/12/13 or Ubuntu 24.04+
- **Collections**: `community.general` (automatically installed as dependency)

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install adicrescenzo.proxmox_backup_server_lxc
```

### From Source

```bash
git clone https://github.com/adicrescenzo/proxmox-backup-server-lxc.git
cd proxmox-backup-server-lxc
ansible-galaxy collection build .
ansible-galaxy collection install adicrescenzo-proxmox_backup_server_lxc-*.tar.gz
```

## Collection Contents

### Roles

| Role | Description | Documentation |
|------|-------------|---------------|
| [`lxc_setup`](roles/lxc_setup/README.md) | Creates and configures LXC containers on Proxmox VE for PBS | [📖 README](roles/lxc_setup/README.md) |
| [`pbs_setup`](roles/pbs_setup/README.md) | Installs and configures Proxmox Backup Server on existing systems | [📖 README](roles/pbs_setup/README.md) |

### Playbooks

| Playbook | Description |
|----------|-------------|
| `pbs.yml` | Complete PBS deployment workflow (LXC + PBS installation) |

## Quick Start

### 1. Create Inventory

Create an inventory file (`inventory.yml`):

```yaml
all:
  children:
    proxmox:
      hosts:
        pve-node1:
          ansible_host: 192.168.1.10
          ansible_user: root
    pbs:
      hosts:
        pbs-server:
          ansible_host: 192.168.1.20
          ansible_user: root
  vars:
    # Proxmox API Configuration
    proxmox_api_token_id: "ansible@pam!deployment"
    proxmox_api_token_secret: "{{ vault_proxmox_api_token }}"
    
    # PBS Configuration
    pbs_lxc_root_password: "{{ vault_pbs_root_password }}"
```

### 2. Basic Deployment

```bash
ansible-playbook -i inventory.yml adicrescenzo.proxmox_backup_server_lxc.pbs
```

## Usage Examples

### Complete PBS Deployment with NFS Storage

```yaml
---
- name: Deploy Proxmox Backup Server with NFS
  hosts: localhost
  gather_facts: false
  vars:
    # LXC Container Configuration
    proxmox_node: "pve-node1"
    proxmox_api_token_id: "ansible@pam!deployment"
    proxmox_api_token_secret: "{{ vault_proxmox_api_token }}"
    lxc_id: 150
    lxc_hostname: "pbs-backup"
    lxc_storage: "local-lvm"
    lxc_cores: 4
    lxc_memory_mb: 4096
    lxc_rootfs_size: 20
    lxc_net0: "name=eth0,ip=192.168.1.150/24,gw=192.168.1.1,bridge=vmbr0"
    lxc_root_password: "{{ vault_pbs_root_password }}"
    
    # PBS Configuration
    pbs_license: "community"
    mount_nfs: true
    nfs_server: "192.168.1.100"
    nfs_share: "/volume1/backups"
    nfs_mount_point: "/mnt/backup-storage"
    
    # TLS Configuration
    pbs_tls_manage: true
    pbs_tls_cert: "{{ vault_pbs_certificate }}"
    pbs_tls_cert_key: "{{ vault_pbs_private_key }}"

  tasks:
    # Create LXC Container
    - name: Setup LXC Container
      include_role:
        name: adicrescenzo.proxmox_backup_server_lxc.lxc_setup
      vars:
        lxc_install: true
        lxc_configure: true
      delegate_to: "{{ proxmox_node }}"

    # Install PBS
    - name: Install and Configure PBS
      include_role:
        name: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_install: true
        pbs_configure: true
      delegate_to: pbs-backup
```

### Enterprise Edition Deployment

```yaml
---
- name: Deploy PBS Enterprise Edition
  hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_install: true
        pbs_configure: true
        pbs_license: "enterprise"
        
        # Custom user configuration
        pbs_user: "backup"
        pbs_user_group: "backup"
        
        # NFS storage
        mount_nfs: true
        nfs_server: "storage.example.com"
        nfs_share: "/backups/pbs"
        nfs_mount_point: "/mnt/enterprise-backups"
        mount_options: "vers=4,rw,hard,intr,timeo=60"
```

### Configuration-Only Deployment

```yaml
---
- name: Configure Existing PBS Installation
  hosts: existing_pbs
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_configure: true  # Only configure, don't install
        
        # Add NFS mount to existing installation
        mount_nfs: true
        nfs_server: "new-storage.example.com"
        nfs_share: "/new-backup-location"
        nfs_mount_point: "/mnt/additional-storage"
        
        # Update TLS certificates
        pbs_tls_manage: true
        pbs_tls_cert: "{{ new_certificate }}"
        pbs_tls_cert_key: "{{ new_private_key }}"
```

## Variable Reference

### Global Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `manage_lxc` | boolean | `true` | Whether to manage LXC container creation |
| `proxmox_host` | string | `groups['proxmox'][0]` | Proxmox host for LXC operations |
| `pbs_host` | string | `groups['pbs']` | Target host for PBS installation |

### LXC Setup Variables

For complete variable reference, see [LXC Setup Role Documentation](roles/lxc_setup/README.md)

**Key Variables:**
- `lxc_install` (boolean, default: `false`) - Create new LXC container
- `lxc_configure` (boolean, default: `false`) - Configure LXC container
- `lxc_id` (integer, **required**) - Unique VMID for container
- `lxc_storage` (string, **required**) - Proxmox storage backend
- `lxc_root_password` (string, **required**) - Root password

### PBS Setup Variables

For complete variable reference, see [PBS Setup Role Documentation](roles/pbs_setup/README.md)

**Key Variables:**
- `pbs_install` (boolean, default: `false`) - Install PBS
- `pbs_configure` (boolean, default: `false`) - Configure PBS
- `pbs_license` (string, default: `"community"`) - License type: `community`/`enterprise`
- `mount_nfs` (boolean, default: `false`) - Enable NFS storage
- `pbs_tls_manage` (boolean, default: `false`) - Manage TLS certificates

## Sample Inventory

### Complete Production Inventory

```yaml
---
all:
  children:
    # Proxmox VE Hosts
    proxmox:
      hosts:
        pve-node1:
          ansible_host: 10.0.1.10
          ansible_user: root
          ansible_ssh_private_key_file: ~/.ssh/proxmox_key
        pve-node2:
          ansible_host: 10.0.1.11
          ansible_user: root
          ansible_ssh_private_key_file: ~/.ssh/proxmox_key
      vars:
        proxmox_api_user: "root@pam"
        proxmox_api_token_id: "ansible@pam!deployment"
        proxmox_api_token_secret: "{{ vault_proxmox_api_token }}"

    # PBS Instances
    pbs:
      hosts:
        pbs-primary:
          ansible_host: 10.0.1.20
          ansible_user: root
          # LXC specific vars
          lxc_id: 120
          lxc_hostname: "pbs-primary"
          lxc_cores: 4
          lxc_memory_mb: 8192
          lxc_rootfs_size: 50
          lxc_net0: "name=eth0,ip=10.0.1.20/24,gw=10.0.1.1,bridge=vmbr0"
          # PBS specific vars
          nfs_server: "storage1.company.com"
          nfs_share: "/backups/pbs-primary"
          nfs_mount_point: "/mnt/primary-backups"
          
        pbs-secondary:
          ansible_host: 10.0.1.21
          ansible_user: root
          # LXC specific vars
          lxc_id: 121
          lxc_hostname: "pbs-secondary"
          lxc_cores: 2
          lxc_memory_mb: 4096
          lxc_rootfs_size: 30
          lxc_net0: "name=eth0,ip=10.0.1.21/24,gw=10.0.1.1,bridge=vmbr0"
          # PBS specific vars
          nfs_server: "storage2.company.com"
          nfs_share: "/backups/pbs-secondary"
          nfs_mount_point: "/mnt/secondary-backups"
      
      vars:
        # Common PBS configuration
        pbs_license: "enterprise"
        pbs_user: "backup"
        pbs_user_group: "backup"
        mount_nfs: true
        mount_options: "vers=4,rw,hard,intr,timeo=60,retrans=3"
        
        # TLS Configuration
        pbs_tls_manage: true
        pbs_tls_cert: "{{ vault_pbs_wildcard_cert }}"
        pbs_tls_cert_key: "{{ vault_pbs_wildcard_key }}"
        
        # Common LXC configuration
        lxc_storage: "ceph-storage"
        lxc_image: "local:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst"
        lxc_privileged: true
        lxc_nesting: true
        lxc_keyctl: true
        lxc_root_password: "{{ vault_pbs_root_password }}"
        
  vars:
    # Global configuration
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    
    # Vault variables (store these in ansible-vault)
    vault_proxmox_api_token: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          # ... encrypted content ...
    vault_pbs_root_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          # ... encrypted content ...
    vault_pbs_wildcard_cert: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          # ... encrypted content ...
    vault_pbs_wildcard_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          # ... encrypted content ...
```

### Simple Development Inventory

```yaml
---
all:
  children:
    proxmox:
      hosts:
        pve-dev:
          ansible_host: 192.168.100.10
          ansible_user: root
    pbs:
      hosts:
        pbs-dev:
          ansible_host: 192.168.100.20
          ansible_user: root
          lxc_id: 200
          lxc_hostname: "pbs-dev"
          
  vars:
    # Development configuration
    proxmox_node: "pve-dev"
    proxmox_api_token_id: "ansible@pam!dev"
    proxmox_api_token_secret: "dev-token-secret"
    
    # Simple LXC config
    lxc_storage: "local-lvm"
    lxc_cores: 2
    lxc_memory_mb: 2048
    lxc_rootfs_size: 20
    lxc_net0: "name=eth0,ip=dhcp,bridge=vmbr0"
    lxc_root_password: "development"
    
    # Community PBS
    pbs_license: "community"
    mount_nfs: false
```

## Advanced Usage

### Deployment with Ansible Vault

1. Create vault file:
```bash
ansible-vault create group_vars/all/vault.yml
```

2. Add sensitive variables:
```yaml
vault_proxmox_api_token: "your-secure-token"
vault_pbs_root_password: "secure-password"
vault_pbs_certificate: |
  -----BEGIN CERTIFICATE-----
  ...
  -----END CERTIFICATE-----
vault_pbs_private_key: |
  -----BEGIN PRIVATE KEY-----
  ...
  -----END PRIVATE KEY-----
```

3. Run playbook with vault:
```bash
ansible-playbook -i inventory.yml --ask-vault-pass playbook.yml
```

### Multiple PBS Instances

```bash
# Deploy to specific group
ansible-playbook -i inventory.yml playbook.yml --limit pbs_primary

# Deploy all instances
ansible-playbook -i inventory.yml playbook.yml

# Configure only (skip installation)
ansible-playbook -i inventory.yml playbook.yml -e "pbs_install=false pbs_configure=true"
```

## Troubleshooting

### Common Issues

1. **LXC Creation Fails**: Verify Proxmox API credentials and node names
2. **PBS Installation Fails**: Check internet connectivity and repository access
3. **NFS Mount Issues**: Verify NFS server accessibility and export permissions
4. **TLS Certificate Errors**: Validate certificate format and file permissions

### Debug Mode

Enable debug output:
```bash
ansible-playbook -i inventory.yml playbook.yml -vvv
```

### Validation

After deployment, verify PBS is running:
```bash
# Check service status
systemctl status proxmox-backup-proxy

# Access web interface
curl -k https://pbs-server-ip:8007
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This collection is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## Support

- 🐛 **Issues**: [GitHub Issues](https://github.com/adicrescenzo/proxmox-backup-server-lxc/issues)
- 📖 **Documentation**: [GitHub Repository](https://github.com/adicrescenzo/proxmox-backup-server-lxc)
- ⭐ **Galaxy**: [Ansible Galaxy](https://galaxy.ansible.com/adicrescenzo/proxmox_backup_server_lxc)

## Changelog

### Version 1.0.0
- Initial release
- LXC container management for Proxmox VE
- Proxmox Backup Server installation and configuration
- NFS storage support
- TLS certificate management
- Community and Enterprise edition support
