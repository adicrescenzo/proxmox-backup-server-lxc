Proxmox Backup Server Setup Role
================================

This Ansible role installs and configures Proxmox Backup Server (PBS) on existing LXC containers or virtual machines. It supports both Community and Enterprise editions, handles NFS mount configuration, and manages TLS certificates for secure connections.

Requirements
------------

- Ansible 2.1 or later
- Target system running Debian (11/12/13) or Ubuntu (24.04+)
- Internet connectivity for package downloads
- Root access on target systems
- For NFS functionality: NFS server with accessible shares
- For Enterprise edition: Valid Proxmox subscription

Role Variables
--------------

### Behavior Control Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `pbs_install` | boolean | `false` | Whether to install Proxmox Backup Server |
| `pbs_configure` | boolean | `false` | Whether to configure Proxmox Backup Server |
| `pbs_delete` | boolean | `false` | Whether to delete Proxmox Backup Server |

### PBS Installation Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `pbs_license` | string | `community` | License type: `community` or `enterprise` |
| `pbs_user` | string | `backup` | System user for PBS service |
| `pbs_user_group` | string | `backup` | System group for PBS service |

### NFS Mount Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `mount_nfs` | boolean | `false` | Whether to configure NFS mount for backups |
| `nfs_server` | string | **Required if mount_nfs=true** | NFS server hostname or IP address |
| `nfs_share` | string | **Required if mount_nfs=true** | NFS share path on the server |
| `nfs_mount_point` | string | **Required if mount_nfs=true** | Local mount point for NFS share |
| `mount_options` | string | `vers=4,nouser,atime,auto,retrans=2,rw,dev,exec` | NFS mount options |

### TLS Certificate Configuration

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `pbs_tls_manage` | boolean | `false` | Whether to manage TLS certificates |
| `pbs_tls_cert` | string | **Required if pbs_tls_manage=true** | TLS certificate content (PEM format) |
| `pbs_tls_cert_key` | string | **Required if pbs_tls_manage=true** | TLS private key content (PEM format) |
| `pbs_tls_cert_path` | string | `/etc/proxmox-backup/proxy.pem` | Path for TLS certificate file |
| `pbs_tls_key_path` | string | `/etc/proxmox-backup/proxy.key` | Path for TLS private key file |

Dependencies
------------

This role has no external dependencies, but requires the following system packages which are automatically installed:

- `proxmox-backup-server` - Main PBS package
- `nfs-common` and `nfs4-acl-tools` - For NFS functionality (when enabled)

Example Playbooks
-----------------

### Install PBS Community Edition

```yaml
- hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_install: true
        pbs_configure: true
        pbs_license: community
```

### Install PBS Enterprise Edition

```yaml
- hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_install: true
        pbs_configure: true
        pbs_license: enterprise
```

### Install PBS with NFS Storage

```yaml
- hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_install: true
        pbs_configure: true
        pbs_license: community
        
        # NFS Configuration
        mount_nfs: true
        nfs_server: "192.168.1.100"
        nfs_share: "/volume1/backups"
        nfs_mount_point: "/mnt/nfs-backups"
        mount_options: "vers=4,rw,hard,intr,timeo=60,retrans=3"
```

### Install PBS with Custom TLS Certificates

```yaml
- hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        pbs_install: true
        pbs_configure: true
        pbs_license: community
        
        # TLS Configuration
        pbs_tls_manage: true
        pbs_tls_cert: |
          -----BEGIN CERTIFICATE-----
          MIIDbTCCAlWgAwIBAgIJAOzX...
          -----END CERTIFICATE-----
        pbs_tls_cert_key: |
          -----BEGIN PRIVATE KEY-----
          MIIEvQIBADANBgkqhkiG9w0B...
          -----END PRIVATE KEY-----
```

### Configure Existing PBS Installation

```yaml
- hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        # Only configure, don't install
        pbs_configure: true
        
        # Add NFS mount to existing installation
        mount_nfs: true
        nfs_server: "storage.example.com"
        nfs_share: "/backups/pbs"
        nfs_mount_point: "/mnt/nfs-backups"
```

### Complete PBS Setup with All Features

```yaml
- hosts: pbs_servers
  become: true
  roles:
    - role: adicrescenzo.proxmox_backup_server_lxc.pbs_setup
      vars:
        # Installation
        pbs_install: true
        pbs_configure: true
        pbs_license: community
        pbs_user: backup
        pbs_user_group: backup
        
        # NFS Storage
        mount_nfs: true
        nfs_server: "{{ backup_nfs_server }}"
        nfs_share: "/volume1/proxmox-backups"
        nfs_mount_point: "/mnt/backup-storage"
        mount_options: "vers=4,rw,hard,intr"
        
        # TLS Security
        pbs_tls_manage: true
        pbs_tls_cert: "{{ vault_pbs_certificate }}"
        pbs_tls_cert_key: "{{ vault_pbs_private_key }}"
```

Usage Notes
-----------

1. **Behavior Control**: You must set at least one of `pbs_install`, `pbs_configure`, or `pbs_delete` to `true`.

2. **Mutual Exclusivity**: `pbs_install` and `pbs_delete` cannot both be `true` at the same time.

3. **License Types**:
   - `community`: Uses the free Proxmox PBS repository
   - `enterprise`: Requires valid Proxmox subscription for access

4. **NFS Requirements**: When `mount_nfs` is `true`, you must provide `nfs_server`, `nfs_share`, and `nfs_mount_point`.

5. **TLS Management**: When `pbs_tls_manage` is `true`, you must provide both `pbs_tls_cert` and `pbs_tls_cert_key`.

6. **System Updates**: The role performs a full system upgrade before installing PBS to ensure compatibility.

7. **Platform Support**: 
   - Debian 11 (Bullseye), 12 (Bookworm), 13 (Trixie)
   - Ubuntu 24.04+ (uses Bookworm repositories)

8. **NFS Mount**: Uses systemd mount units instead of `/etc/fstab` for better LXC container compatibility.

Post-Installation
-----------------

After successful installation, Proxmox Backup Server will be accessible via:

- **Web Interface**: `https://<server-ip>:8007`
- **Default Credentials**: Set up during first web access
- **Service Management**: `systemctl status proxmox-backup-proxy`

For NFS-mounted storage, ensure the mount point has proper permissions for the PBS user.

License
-------

MIT

Author Information
------------------

Created by adicrescenzo for automated Proxmox Backup Server deployment.
