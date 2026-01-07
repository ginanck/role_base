<!-- DOCSIBLE START -->
# Ansible Role: role_base


role_base to configure base settings


## Table of Contents

- [Requirements](#requirements)
- [Dependencies](#dependencies)
- [Role Variables](#role-variables)
- [Task Overview](#task-overview)
- [Example Playbook](#example-playbook)
- [Documentation Maintenance](#documentation-maintenance)
- [License](#license)
- [Author Information](#author-information)

## Requirements



- Ansible >= 2.9


- Supported platforms:
  - Ubuntu (jammy, noble)
  - Debian (bullseye, bookworm)
  - AlmaLinux (9, 10)
  - RockyLinux (9.0, 10)



## Dependencies


This role requires the following roles and collections:




  
    
  

  
    
  

  
    
  





**Collections:**

- `community.docker` (>= 4.8.1)

- `community.general` (>= 6.6.1)

- `ansible.posix` (>= 1.5.4)



To install all dependencies:
```bash
ansible-galaxy install -r meta/install_requirements.yml
```


## Role Variables



### File: `defaults/main.yml`

| Variable | Type | Default Value | Description |
|----------|------|---------------|-------------|
| [`base_domain`](defaults/main.yml#L4) | str | `internal.guru` | None |
| [`base_hostname`](defaults/main.yml#L5) | str | `` | None |
| [`base_hostname_configured`](defaults/main.yml#L6) | bool | `True` | None |
| [`base_ca_install_enabled`](defaults/main.yml#L9) | bool | `True` | None |
| [`base_ca_script_url`](defaults/main.yml#L10) | str | `http://ca.internal.guru/scripts/install-linux.sh` | None |
| [`base_configure_cloud_init`](defaults/main.yml#L12) | bool | `True` | None |
| [`base_swap_disabled`](defaults/main.yml#L13) | bool | `False` | None |
| [`base_apply_os_patches`](defaults/main.yml#L16) | bool | `True` | None |
| [`base_apply_kernel_patches`](defaults/main.yml#L17) | bool | `True` | None |
| [`base_apply_security_patches`](defaults/main.yml#L18) | bool | `True` | None |
| [`base_reboot_after_patches`](defaults/main.yml#L20) | bool | `False` | None |
| [`base_reboot_timeout`](defaults/main.yml#L21) | int | `600` | None |
| [`base_disable_gpg_check`](defaults/main.yml#L22) | bool | `True` | None |
| [`base_security_method`](defaults/main.yml#L25) | str | `unattended-upgrades` | None |
| [`base_security_auto_reboot`](defaults/main.yml#L26) | bool | `False` | None |
| [`base_security_auto_reboot_time`](defaults/main.yml#L27) | str | `02:00` | None |
| [`base_security_remove_unused_deps`](defaults/main.yml#L28) | bool | `True` | None |
| [`base_security_auto_updates_daily`](defaults/main.yml#L29) | bool | `False` | None |
| [`base_hostname_entries`](defaults/main.yml#L31) | list | `[]` | None |
| [`base_resolv_conf_managed`](defaults/main.yml#L39) | bool | `True` | None |
| [`base_resolv_nameserver_entries`](defaults/main.yml#L40) | list | See below | None |
| [`base_resolv_nameserver_search_domains`](defaults/main.yml#L43) | list | See below | None |
| [`base_resolv_nameserver_resolv_options`](defaults/main.yml#L46) | list | See below | None |
| [`base_default_packages`](defaults/main.yml#L51) | list | See below | None |
| [`base_additional_packages`](defaults/main.yml#L65) | list | `[]` | None |
| [`base_timezone`](defaults/main.yml#L68) | str | `Europe/Helsinki` | None |
| [`base_chrony_keys`](defaults/main.yml#L69) | list | `[]` | None |
| [`base_chrony_config`](defaults/main.yml#L70) | dict | See below | None |
| [`base_lvm_disks`](defaults/main.yml#L101) | list | `[]` | None |

#### `base_resolv_nameserver_entries`

```yaml
- 
```

#### `base_resolv_nameserver_search_domains`

```yaml
- 
```

#### `base_resolv_nameserver_resolv_options`

```yaml
- 
- 
```

#### `base_default_packages`

```yaml
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
```

#### `base_chrony_config`

```yaml
```




## Task Overview


This role performs the following tasks:


### File: `tasks/resolve.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update resolv.conf](tasks/resolve.yml#L) | ansible.builtin.template | No | N/A |




### File: `tasks/cloud_init.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Ensure cloud-init config directory exists](tasks/cloud_init.yml#L) | ansible.builtin.file | No | N/A |
| [Set preserve_hostname to true in cloud-init config](tasks/cloud_init.yml#L) | ansible.builtin.lineinfile | No | N/A |
| [Remove update_etc_hosts from cloud_init_modules](tasks/cloud_init.yml#L) | ansible.builtin.lineinfile | No | N/A |
| [Restart cloud-init](tasks/cloud_init.yml#L) | ansible.builtin.systemd | Yes | N/A |




### File: `tasks/os_patches.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Apply security patches based on package manager](tasks/os_patches.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Apply OS patches based on package manager](tasks/os_patches.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Apply kernel patches based on package manager](tasks/os_patches.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Check if reboot is required (Debian/Ubuntu)](tasks/os_patches.yml#L) | ansible.builtin.stat | Yes | N/A |
| [Check if reboot is required (RedHat/CentOS/Rocky)](tasks/os_patches.yml#L) | ansible.builtin.shell | Yes | N/A |
| [Set reboot required fact](tasks/os_patches.yml#L) | ansible.builtin.set_fact | No | N/A |
| [Reboot system if required and configured](tasks/os_patches.yml#L) | ansible.builtin.reboot | Yes | N/A |
| [Display patch results](tasks/os_patches.yml#L) | ansible.builtin.debug | No | N/A |




### File: `tasks/chronyd.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Generate /etc/chrony.conf](tasks/chronyd.yml#L) | ansible.builtin.template | Yes | N/A |
| [Generate /etc/chrony.keys](tasks/chronyd.yml#L) | ansible.builtin.template | Yes | N/A |
| [Create systemd drop-in directory for chrony (container only)](tasks/chronyd.yml#L) | ansible.builtin.file | Yes | N/A |
| [Force -x option for chrony in containers (Ubuntu 20.04 workaround)](tasks/chronyd.yml#L) | ansible.builtin.copy | Yes | N/A |
| [Reload systemd daemon](tasks/chronyd.yml#L) | ansible.builtin.systemd | Yes | N/A |
| [Start chrony](tasks/chronyd.yml#L) | ansible.builtin.systemd | Yes | N/A |




### File: `tasks/swap.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Disable swap at runtime](tasks/swap.yml#L) | ansible.builtin.command | No | N/A |
| [Disable swap permanently (in /etc/fstab)](tasks/swap.yml#L) | ansible.builtin.replace | No | N/A |




### File: `tasks/selinux.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Check if SELinux is available](tasks/selinux.yml#L) | ansible.builtin.stat | No | N/A |
| [Disable SELinux immediately (if installed)](tasks/selinux.yml#L) | ansible.builtin.command | Yes | N/A |
| [Disable SELinux permanently (if installed)](tasks/selinux.yml#L) | ansible.builtin.lineinfile | Yes | N/A |
| [Check if AppArmor is installed (Debian/Ubuntu)](tasks/selinux.yml#L) | ansible.builtin.stat | Yes | N/A |
| [Disable AppArmor (if installed on Debian/Ubuntu)](tasks/selinux.yml#L) | ansible.builtin.service | Yes | N/A |




### File: `tasks/timezone.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Set timezone on nodes](tasks/timezone.yml#L) | community.general.timezone | No | N/A |




### File: `tasks/firewall.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Gather OS facts](tasks/firewall.yml#L) | ansible.builtin.setup | No | N/A |
| [Stop and disable firewalld service](tasks/firewall.yml#L) | ansible.builtin.systemd | Yes | N/A |
| [Stop and disable ufw service](tasks/firewall.yml#L) | ansible.builtin.systemd | Yes | N/A |




### File: `tasks/grow-partition.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Extract base disk and partition number for partitioned disks](tasks/grow-partition.yml#L) | ansible.builtin.set_fact | No | N/A |
| [Check if partition needs to grow (dry-run)](tasks/grow-partition.yml#L) | ansible.builtin.command | No | N/A |
| [Grow partition only if needed](tasks/grow-partition.yml#L) | ansible.builtin.command | Yes | N/A |




### File: `tasks/lvm.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Detect disk type (partition vs whole disk)](tasks/lvm.yml#L) | ansible.builtin.set_fact | No | N/A |
| [Extend partition-based disk](tasks/lvm.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Create a Volume Group](tasks/lvm.yml#L) | community.general.lvg | No | N/A |
| [Create a Logical Volume](tasks/lvm.yml#L) | community.general.lvol | No | N/A |
| [Check if filesystem exists](tasks/lvm.yml#L) | ansible.builtin.command | No | N/A |
| [Make Filesystem](tasks/lvm.yml#L) | community.general.filesystem | Yes | N/A |
| [Create mount point directories](tasks/lvm.yml#L) | ansible.builtin.file | No | N/A |
| [Mount Filesystem](tasks/lvm.yml#L) | ansible.posix.mount | No | N/A |




### File: `tasks/main.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Include Base Vars](tasks/main.yml#L) | ansible.builtin.include_vars | No | N/A |
| [Install Prerequisite Packages](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |
| [Apply OS Patches](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Setup Epel-Repo](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Install CA Certificate](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Disable Cloud-Init manage_hostname](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Setup resolve.conf](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Setup Hostname of the Nodes](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Setup Hosts](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |
| [Setup Timezone](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |
| [Setup Chrony](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |
| [Disable Selinux](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |
| [Disable Firewall](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |
| [Disable Swap](tasks/main.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Setup LVM Disk](tasks/main.yml#L) | ansible.builtin.include_tasks | No | N/A |




### File: `tasks/certificate.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Download CA certificate installation script](tasks/certificate.yml#L) | ansible.builtin.get_url | No | N/A |
| [Check if downloaded file exists and has content](tasks/certificate.yml#L) | ansible.builtin.stat | No | N/A |
| [Check first few lines of downloaded file for debugging](tasks/certificate.yml#L) | ansible.builtin.shell | Yes | N/A |
| [Display download result for debugging](tasks/certificate.yml#L) | ansible.builtin.debug | No | N/A |
| [Check if CA script download was successful](tasks/certificate.yml#L) | ansible.builtin.debug | Yes | N/A |
| [Validate downloaded script is executable](tasks/certificate.yml#L) | ansible.builtin.shell | Yes | N/A |
| [Display script validation result](tasks/certificate.yml#L) | ansible.builtin.debug | Yes | N/A |
| [Execute CA certificate installation script](tasks/certificate.yml#L) | ansible.builtin.shell | Yes | N/A |
| [Remove temporary installation script](tasks/certificate.yml#L) | ansible.builtin.file | Yes | N/A |
| [Display certificate installation output](tasks/certificate.yml#L) | ansible.builtin.debug | Yes | N/A |




### File: `tasks/hostname.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Set hostname on the node](tasks/hostname.yml#L) | ansible.builtin.hostname | No | N/A |




### File: `tasks/hosts.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Gather network facts](tasks/hosts.yml#L) | ansible.builtin.setup | No | N/A |
| [Check if /etc/hosts is accessible](tasks/hosts.yml#L) | ansible.builtin.stat | No | N/A |
| [Update hostname entries in /etc/hosts](tasks/hosts.yml#L) | ansible.builtin.template | No | N/A |




### File: `tasks/epel/RedHat.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Setup EPEL repo](tasks/epel/RedHat.yml#L) | ansible.builtin.dnf | No | N/A |




### File: `tasks/patches/security/apt-sources.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (apt)](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.apt | No | N/A |
| [Create security sources directory](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.file | No | N/A |
| [Create security-only sources list for Debian](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.template | Yes | N/A |
| [Create security-only sources list for Ubuntu](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.template | Yes | N/A |
| [Update package cache with security sources](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.apt | No | N/A |
| [Apply security updates for Debian systems](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.apt | Yes | N/A |
| [Apply security updates for Ubuntu systems](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.apt | Yes | N/A |
| [Remove unused packages after security updates](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.apt | Yes | N/A |
| [Display security update results](tasks/patches/security/apt-sources.yml#L) | ansible.builtin.debug | No | N/A |




### File: `tasks/patches/security/apt-unattended.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (apt)](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.apt | No | N/A |
| [Install required packages for security updates](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.apt | No | N/A |
| [Configure unattended-upgrades for security updates only](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.template | No | N/A |
| [Configure unattended-upgrades automatic updates](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.template | No | N/A |
| [Enable unattended-upgrades service](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.systemd | No | N/A |
| [Run dry-run to check available security updates](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.command | No | N/A |
| [Check if security updates are available](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.set_fact | No | N/A |
| [Display available security updates](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.debug | Yes | N/A |
| [Display no security updates message](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.debug | Yes | N/A |
| [Apply security updates using unattended-upgrades](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.command | Yes | N/A |
| [Clean up package cache after security updates](tasks/patches/security/apt-unattended.yml#L) | ansible.builtin.apt | No | N/A |




### File: `tasks/patches/security/apt.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Apply security updates using unattended-upgrades method](tasks/patches/security/apt.yml#L) | ansible.builtin.include_tasks | Yes | N/A |
| [Apply security updates using apt-sources method](tasks/patches/security/apt.yml#L) | ansible.builtin.include_tasks | Yes | N/A |




### File: `tasks/patches/security/yum.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (yum)](tasks/patches/security/yum.yml#L) | ansible.builtin.yum | No | N/A |
| [Apply security updates only (yum)](tasks/patches/security/yum.yml#L) | ansible.builtin.yum | No | N/A |




### File: `tasks/patches/security/dnf.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (dnf)](tasks/patches/security/dnf.yml#L) | ansible.builtin.dnf | No | N/A |
| [Apply security updates only (dnf)](tasks/patches/security/dnf.yml#L) | ansible.builtin.dnf | No | N/A |




### File: `tasks/patches/os/apt.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (apt)](tasks/patches/os/apt.yml#L) | ansible.builtin.apt | No | N/A |
| [Get list of upgradable packages excluding kernel packages](tasks/patches/os/apt.yml#L) | ansible.builtin.shell | No | N/A |
| [Apply OS updates excluding kernel packages (apt)](tasks/patches/os/apt.yml#L) | ansible.builtin.apt | Yes | N/A |




### File: `tasks/patches/os/yum.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (yum)](tasks/patches/os/yum.yml#L) | ansible.builtin.yum | No | N/A |
| [Apply all available OS updates excluding kernel (yum)](tasks/patches/os/yum.yml#L) | ansible.builtin.yum | No | N/A |




### File: `tasks/patches/os/dnf.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (dnf)](tasks/patches/os/dnf.yml#L) | ansible.builtin.dnf | No | N/A |
| [Apply all available OS updates excluding kernel (dnf)](tasks/patches/os/dnf.yml#L) | ansible.builtin.dnf | No | N/A |




### File: `tasks/patches/kernel/apt.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Update package cache (apt)](tasks/patches/kernel/apt.yml#L) | ansible.builtin.apt | No | N/A |
| [Apply kernel updates (apt)](tasks/patches/kernel/apt.yml#L) | ansible.builtin.apt | No | N/A |




### File: `tasks/patches/kernel/yum.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Apply kernel updates (yum)](tasks/patches/kernel/yum.yml#L) | ansible.builtin.yum | No | N/A |




### File: `tasks/patches/kernel/dnf.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Apply kernel updates (dnf)](tasks/patches/kernel/dnf.yml#L) | ansible.builtin.dnf | No | N/A |




### File: `tasks/package/apt.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Install prerequisites (Debian family)](tasks/package/apt.yml#L) | ansible.builtin.apt | No | N/A |




### File: `tasks/package/yum.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Install prerequisites (RedHat family with yum)](tasks/package/yum.yml#L) | ansible.builtin.yum | No | N/A |




### File: `tasks/package/dnf.yml`

| Task Name | Module | Has Conditions | Line |
|-----------|--------|----------------|------|
| [Install prerequisites (RedHat family with dnf)](tasks/package/dnf.yml#L) | ansible.builtin.dnf | No | N/A |






## Example Playbook

```yaml
---
- hosts: all
  become: yes
  roles:
    - role: role_base

      vars:
        base_domain: internal.guru
        base_hostname: 
        base_hostname_configured: True

```

## Documentation Maintenance

### Updating Dependencies

1. **Update** `meta/main.yml`:
   ```yaml
   documented_requirements:
     - src: https://github.com/user/role.git
       version: master
     - name: collection.name
       version: 1.0.0
   ```

2. **Sync** `meta/install_requirements.yml` with the same requirements

3. **Regenerate** documentation:
   ```bash
   pre-commit run --all-files
   ```

### Template Updates

- Edit `.docsible_template.md` for structure changes
- Test with: `docsible --role . --md-template .docsible_template.md -nob -com -tl`
- Commit both template and generated README.md

### Quick Checklist

When updating dependencies:
- [ ] Add to `meta/main.yml` → `documented_requirements`
- [ ] Add to `meta/install_requirements.yml`
- [ ] Run `pre-commit run --all-files`
- [ ] Verify generated README.md
- [ ] Commit all changes

## License


license (GPL-2.0-or-later, MIT, etc)


## Author Information


**Author:** gkorkmaz




**GitHub:** [gkorkmaz](https://github.com/gkorkmaz)

---
*This documentation was automatically generated using [docsible](https://github.com/zbohm/docsible).*
<!-- DOCSIBLE END -->
