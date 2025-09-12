Role Base
=========

A comprehensive base role for Linux systems that handles fundamental system configuration, package management, and OS patching.

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
- `base_reboot_after_patches`: (bool, default: false) Whether to reboot system after patches if required
- `base_reboot_timeout`: (int, default: 600) Timeout in seconds for reboot operation
- `base_security_updates_only`: (bool, default: false) Apply only security updates (RHEL-based systems only)

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
      base_reboot_after_patches: true
      base_additional_packages:
        - htop
        - tree
        - jq
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

This role includes comprehensive OS patch management capabilities:

1. **Automatic Updates**: Updates all packages to their latest versions
2. **Cache Management**: Handles package cache updates for all supported package managers
3. **Reboot Detection**: Automatically detects when a reboot is required after updates
4. **Optional Reboot**: Can automatically reboot the system when required (disabled by default)
5. **Cross-Platform**: Supports apt (Debian/Ubuntu), yum, and dnf (RHEL/CentOS/Rocky) package managers

The patch management runs after package installation but before other system configuration tasks to ensure a clean, updated base system.

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
