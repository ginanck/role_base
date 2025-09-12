Role Base
=========

A comprehensive base role for Linux systems that handles fundamSecurity-only updates for compliance environments:

```yaml
- hosts: compliance_servers
  roles:
    - role: role_base
      base_apply_security_patches: true  # Only security updates
      base_apply_os_patches: false
      base_apply_kernel_patches: false
```

Advanced Ubuntu/Debian security configuration with unattended-upgrades:

```yaml
- hosts: ubuntu_servers
  roles:
    - role: role_base
      base_apply_security_patches: true
      base_security_method: "unattended-upgrades"  # Default method
      base_security_auto_updates_daily: true       # Enable daily automatic security updates
      base_security_auto_reboot: true              # Allow automatic reboot for security updates
      base_security_auto_reboot_time: "03:00"      # Schedule reboot at 3 AM
```

Alternative Ubuntu/Debian security with direct apt-sources:

```yaml
- hosts: ubuntu_servers
  roles:
    - role: role_base
      base_apply_security_patches: true
      base_security_method: "apt-sources"          # Direct apt-get approach
      base_security_remove_unused_deps: true
```

Disable kernel patching for VMs that shouldn't be rebooted: configuration, package management, and OS patching.

Requirements
------------

This role requires Ansible 2.9+ and supports the following operating systems:
- RHEL/CentOS/Rocky Linux 7+
- Ubuntu 18.04+
- Debian 9+

Role Variables
--------------

### OS Patch Management
- `base_apply_os_patches`: (bool, default: true) Whether to apply OS patches/updates
- `base_apply_kernel_patches`: (bool, default: true) Whether to apply kernel patches (requires reboot)
- `base_apply_security_patches`: (bool, default: false) Whether to apply only security patches
- `base_reboot_after_patches`: (bool, default: false) Whether to reboot system after patches if required
- `base_reboot_timeout`: (int, default: 600) Timeout in seconds for reboot operation

### Security Update Configuration (Debian/Ubuntu)
- `base_security_method`: (string, default: "unattended-upgrades") Method for security updates ("unattended-upgrades" or "apt-sources")
- `base_security_auto_reboot`: (bool, default: false) Allow automatic reboot after security updates
- `base_security_auto_reboot_time`: (string, default: "02:00") Time for automatic reboot if enabled
- `base_security_remove_unused_deps`: (bool, default: true) Remove unused dependencies after updates
- `base_security_auto_updates_daily`: (bool, default: false) Enable daily automatic security updates

### Package Management
- `base_default_packages`: (list) List of default packages to install
- `base_additional_packages`: (list) Additional packages to install beyond defaults
- `base_os_specific_packages`: (list) OS-specific packages (defined in vars files)

### System Configuration
- `base_hostname`: (string) Hostname to set for the system
- `base_hostname_configured`: (bool, default: true) Whether to configure hostname
- `base_domain`: (string, default: "internal.guru") Domain name for the system
- `base_timezone`: (string, default: "Europe/Helsinki") System timezone
- `base_swap_disabled`: (bool, default: false) Whether to disable swap

### Network Configuration
- `base_resolv_conf_managed`: (bool, default: true) Whether to manage resolv.conf
- `base_resolv_nameserver_entries`: (list) DNS nameserver entries
- `base_resolv_nameserver_search_domains`: (list) DNS search domains

### Security & Services
- `base_ca_install_enabled`: (bool, default: true) Whether to install CA certificates
- `base_configure_cloud_init`: (bool, default: true) Whether to configure cloud-init

Dependencies
------------

No external role dependencies.

Example Playbook
----------------

Basic usage with OS patching enabled:

```yaml
- hosts: servers
  roles:
    - role: role_base
      base_apply_os_patches: true
      base_apply_kernel_patches: true
      base_reboot_after_patches: true
      base_additional_packages:
        - htop
        - tree
        - jq
```

Disable kernel patching for VMs that shouldn't be rebooted:

```yaml
- hosts: database_servers
  roles:
    - role: role_base
      base_apply_os_patches: true
      base_apply_kernel_patches: false  # Skip kernel updates
      base_reboot_after_patches: false
```

Security-only updates for compliance environments:

```yaml
- hosts: compliance_servers
  roles:
    - role: role_base
      base_apply_security_patches: true  # Only security updates
      base_apply_os_patches: false
      base_apply_kernel_patches: false
```

Disable OS patching for specific environments:

```yaml
- hosts: production_servers
  roles:
    - role: role_base
      base_apply_os_patches: false
      base_hostname: "{{ inventory_hostname }}"
```

OS Patch Management
-------------------

This role includes comprehensive OS patch management capabilities with clear separation between OS patches and kernel patches:

### Patch Management Structure
The role organizes patches into three categories:
- **Security Patches** (`tasks/patches/security/`): Security-only updates using native tools
  - **RHEL/CentOS**: Uses `yum/dnf` with `security: true` parameter
  - **Debian/Ubuntu**: Two methods available:
    - `unattended-upgrades` (default): Official Ubuntu/Debian recommendation
    - `apt-sources`: Direct apt-get with security-only source lists
- **OS Patches** (`tasks/patches/os/`): Updates for applications, libraries, and system packages (excluding kernel)
- **Kernel Patches** (`tasks/patches/kernel/`): Updates specifically for kernel packages

### Features

1. **Separated Patch Management**: Clear distinction between OS patches and kernel patches
2. **Granular Control**: Apply OS patches while optionally skipping kernel updates
3. **Automatic Updates**: Updates all packages to their latest versions
4. **Cache Management**: Handles package cache updates for all supported package managers
5. **Reboot Detection**: Automatically detects when a reboot is required after updates
6. **Optional Reboot**: Can automatically reboot the system when required (disabled by default)
7. **Cross-Platform**: Supports apt (Debian/Ubuntu), yum, and dnf (RHEL/CentOS/Rocky) package managers
8. **Security-Only Updates**: Option to apply only security updates as a separate patch category

### Patch Management Workflow
1. **Security Patches**: Applied first if enabled (mutually exclusive with other patch types)
2. **OS Patches**: Applied if security patches are not enabled, excludes kernel packages
3. **Kernel Patches**: Applied separately if enabled and security patches are not enabled
3. **Reboot Detection**: Intelligently determines if a reboot is required
4. **Optional Reboot**: Performs reboot if configured and required

The patch management runs after package installation but before other system configuration tasks to ensure a clean, updated base system.

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
