LXC Setup Role for Proxmox Backup Server
=========================================

This Ansible role sets up and configures LXC containers on Proxmox VE for use as Proxmox Backup Server (PBS). It can create, configure, or delete LXC containers with the necessary features enabled for PBS operation.

Requirements
------------

- Ansible 2.1 or later
- `community.general` collection (for the `proxmox` module)
- A Proxmox VE server with API access enabled
- Valid Proxmox API credentials (API token or username/password)
- SSH access to the Proxmox host for container configuration tasks

Role Variables
--------------

### Mandatory Variables

These variables must be defined when using this role:

| Variable | Type | Description |
|----------|------|-------------|
| `proxmox_node` | string | The Proxmox node name where the LXC container will be created |
| `proxmox_api_token_id` | string | Proxmox API token ID for authentication |
| `proxmox_api_token_secret` | string | Proxmox API token secret for authentication |
| `lxc_id` | integer | Unique VMID for the LXC container (must be > 0) |
| `lxc_root_password` | string | Root password for the LXC container |

### Behavior Control Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `lxc_install` | boolean | `false` | Whether to create a new LXC container |
| `lxc_configure` | boolean | `false` | Whether to configure an existing LXC container |
| `lxc_delete` | boolean | `false` | Whether to delete an existing LXC container |

### Proxmox API Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `proxmox_api_host` | string | `groups['proxmox'][0]` | Proxmox API host address |
| `proxmox_api_user` | string | `root@pam` | Proxmox API user for authentication |

### LXC Container Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `lxc_hostname` | string | `pbs` | Hostname for the LXC container |
| `lxc_image` | string | `local:vztmpl/ubuntu-24.04-standard_24.04-2_amd64.tar.zst` | OS template for the container |
| `lxc_cores` | integer | `2` | Number of CPU cores allocated to the container |
| `lxc_memory_mb` | integer | `2048` | Memory allocation in MB |
| `lxc_swap_mb` | integer | `0` | Swap memory allocation in MB |
| `lxc_rootfs_size` | integer | `8` | Root filesystem size in GB |
| `lxc_storage` | string | **Required** | Proxmox storage backend for the container |
| `lxc_net0` | string | `name=eth0,ip=dhcp,bridge=vmbr1` | Network interface configuration |
| `lxc_privileged` | boolean | `true` | Whether to create a privileged container |
| `lxc_ssh_public_key` | string | `~/.ssh/id_rsa.pub` | Path to SSH public key file |

### LXC Features Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `lxc_nesting` | boolean | `true` | Enable container nesting feature |
| `lxc_keyctl` | boolean | `true` | Enable keyctl feature |

Dependencies
------------

This role requires the `community.general` collection for the Proxmox module. Install it using:

```bash
ansible-galaxy collection install community.general
```

Example Playbook
----------------

### Create a new LXC container for PBS

```yaml
- hosts: proxmox
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.lxc_setup
      vars:
        # Mandatory variables
        proxmox_node: "pve-node1"
        proxmox_api_token_id: "root@pam!ansible"
        proxmox_api_token_secret: "your-api-token-secret"
        lxc_id: 150
        lxc_root_password: "secure-password"
        lxc_storage: "local-lvm"
        
        # Behavior control
        lxc_install: true
        lxc_configure: true
        
        # Optional customization
        lxc_hostname: "pbs-backup"
        lxc_cores: 4
        lxc_memory_mb: 4096
        lxc_rootfs_size: 20
        lxc_net0: "name=eth0,ip=192.168.1.100/24,gw=192.168.1.1,bridge=vmbr0"
```

### Configure an existing LXC container

```yaml
- hosts: proxmox
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.lxc_setup
      vars:
        proxmox_node: "pve-node1"
        proxmox_api_token_id: "root@pam!ansible"
        proxmox_api_token_secret: "your-api-token-secret"
        lxc_id: 150
        lxc_root_password: "secure-password"
        
        # Only configure existing container
        lxc_configure: true
```

### Delete an LXC container

```yaml
- hosts: proxmox
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.lxc_setup
      vars:
        proxmox_node: "pve-node1"
        proxmox_api_token_id: "root@pam!ansible"
        proxmox_api_token_secret: "your-api-token-secret"
        lxc_id: 150
        lxc_root_password: "secure-password"
        lxc_storage: "local-lvm"
        
        # Delete the container
        lxc_delete: true
```

Usage Notes
-----------

1. **Behavior Control**: You must set at least one of `lxc_install`, `lxc_configure`, or `lxc_delete` to `true`.
2. **Mutual Exclusivity**: `lxc_install` and `lxc_delete` cannot both be `true` at the same time.
3. **Container Features**: The role automatically enables nesting and keyctl features required for PBS operation.
4. **SSH Key**: Ensure the SSH public key file exists at the specified path for container access.

License
-------

MIT

Author Information
------------------

Created by adicrescenzo for managing Proxmox Backup Server LXC containers.
