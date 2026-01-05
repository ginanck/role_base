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
  - Ubuntu (focal, jammy, noble)
  - Debian (buster, bullseye, bookworm)
  - EL (7, 8, 9)
  - Rocky (8.0, 9.0)



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

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `base_domain` | `internal.guru` | None |
| `base_hostname` | `` | None |
| `base_hostname_configured` | `True` | None |
| `base_ca_install_enabled` | `True` | None |
| `base_ca_script_url` | `http://ca.internal.guru/scripts/install-linux.sh` | None |
| `base_configure_cloud_init` | `True` | None |
| `base_swap_disabled` | `False` | None |
| `base_apply_os_patches` | `True` | None |
| `base_apply_kernel_patches` | `True` | None |
| `base_apply_security_patches` | `True` | None |
| `base_reboot_after_patches` | `False` | None |
| `base_reboot_timeout` | `600` | None |
| `base_disable_gpg_check` | `True` | None |
| `base_security_method` | `unattended-upgrades` | None |
| `base_security_auto_reboot` | `False` | None |
| `base_security_auto_reboot_time` | `02:00` | None |
| `base_security_remove_unused_deps` | `True` | None |
| `base_security_auto_updates_daily` | `False` | None |
| `base_hostname_entries` | `[]` | None |
| `base_resolv_conf_managed` | `True` | None |
| `base_resolv_nameserver_entries` | `[]` | None |
| `base_resolv_nameserver_entries.0` | `172.16.2.21` | None |
| `base_resolv_nameserver_search_domains` | `[]` | None |
| `base_resolv_nameserver_search_domains.0` | `.` | None |
| `base_resolv_nameserver_resolv_options` | `[]` | None |
| `base_resolv_nameserver_resolv_options.0` | `edns0` | None |
| `base_resolv_nameserver_resolv_options.1` | `trust-ad` | None |
| `base_default_packages` | `[]` | None |
| `base_default_packages.0` | `vim` | None |
| `base_default_packages.1` | `net-tools` | None |
| `base_default_packages.2` | `tar` | None |
| `base_default_packages.3` | `unzip` | None |
| `base_default_packages.4` | `gzip` | None |
| `base_default_packages.5` | `telnet` | None |
| `base_default_packages.6` | `chrony` | None |
| `base_default_packages.7` | `wget` | None |
| `base_default_packages.8` | `curl` | None |
| `base_default_packages.9` | `llvm` | None |
| `base_default_packages.10` | `lvm2` | None |
| `base_default_packages.11` | `git` | None |
| `base_additional_packages` | `[]` | None |
| `base_timezone` | `Europe/Helsinki` | None |
| `base_chrony_keys` | `[]` | None |
| `base_chrony_config` | `{}` | None |
| `base_chrony_config.server` | `{}` | None |
| `base_chrony_config.server.param` | `iburst` | None |
| `base_chrony_config.server.name` | `[]` | None |
| `base_chrony_config.server.name.0` | `0.fi.pool.ntp.org` | None |
| `base_chrony_config.server.name.1` | `1.fi.pool.ntp.org` | None |
| `base_chrony_config.server.name.2` | `2.fi.pool.ntp.org` | None |
| `base_chrony_config.server.name.3` | `3.fi.pool.ntp.org` | None |
| `base_chrony_config.sourcedir` | `/run/chrony-dhcp` | None |
| `base_chrony_config.driftfile` | `/var/lib/chrony/drift` | None |
| `base_chrony_config.makestep` | `1.0 3` | None |
| `base_chrony_config.rtcsync` | `True` | None |
| `base_chrony_config.hwtimestamp` | `*` | None |
| `base_chrony_config.minsources` | `2` | None |
| `base_chrony_config.local` | `{}` | None |
| `base_chrony_config.local.stratum` | `10` | None |
| `base_chrony_config.authselectmode` | `require` | None |
| `base_chrony_config.keyfile` | `/etc/chrony.keys` | None |
| `base_chrony_config.ntsdumpdir` | `/var/lib/chrony` | None |
| `base_chrony_config.leapsecmode` | `slew` | None |
| `base_chrony_config.leapsectz` | `right/UTC` | None |
| `base_chrony_config.logdir` | `/var/log/chrony` | None |
| `base_chrony_config.log` | `{}` | None |
| `base_chrony_config.log.measurements` | `True` | None |
| `base_chrony_config.log.statistics` | `True` | None |
| `base_chrony_config.log.tracking` | `True` | None |
| `base_lvm_disks` | `[]` | None |




## Task Overview


This role performs the following tasks:


### `resolve.yml`


- **Update resolv.conf**


### `cloud_init.yml`


- **Ensure cloud-init config directory exists**
- **Set preserve_hostname to true in cloud-init config**
- **Remove update_etc_hosts from cloud_init_modules**
- **Restart cloud-init**


### `os_patches.yml`


- **Apply security patches based on package manager**
- **Apply OS patches based on package manager**
- **Apply kernel patches based on package manager**
- **Check if reboot is required (Debian/Ubuntu)**
- **Check if reboot is required (RedHat/CentOS/Rocky)**
- **Set reboot required fact**
- **Reboot system if required and configured**
- **Display patch results**


### `chronyd.yml`


- **Generate /etc/chrony.conf**
- **Generate /etc/chrony.keys**
- **Create systemd drop-in directory for chrony (container only)**
- **Force -x option for chrony in containers (Ubuntu 20.04 workaround)**
- **Reload systemd daemon**
- **Start chrony**


### `swap.yml`


- **Disable swap at runtime**
- **Disable swap permanently (in /etc/fstab)**


### `selinux.yml`


- **Check if SELinux is available**
- **Disable SELinux immediately (if installed)**
- **Disable SELinux permanently (if installed)**
- **Check if AppArmor is installed (Debian/Ubuntu)**
- **Disable AppArmor (if installed on Debian/Ubuntu)**


### `timezone.yml`


- **Set timezone on nodes**


### `firewall.yml`


- **Gather OS facts**
- **Stop and disable firewalld service**
- **Stop and disable ufw service**


### `grow-partition.yml`


- **Extract base disk and partition number for partitioned disks**
- **Check if partition needs to grow (dry-run)**
- **Grow partition only if needed**


### `lvm.yml`


- **Detect disk type (partition vs whole disk)**
- **Extend partition-based disk**
- **Create a Volume Group**
- **Create a Logical Volume**
- **Check if filesystem exists**
- **Make Filesystem**
- **Create mount point directories**
- **Mount Filesystem**


### `main.yml`


- **Include Base Vars**
- **Install Prerequisite Packages**
- **Apply OS Patches**
- **Setup Epel-Repo**
- **Install CA Certificate**
- **Disable Cloud-Init manage_hostname**
- **Setup resolve.conf**
- **Setup Hostname of the Nodes**
- **Setup Hosts**
- **Setup Timezone**
- **Setup Chrony**
- **Disable Selinux**
- **Disable Firewall**
- **Disable Swap**
- **Setup LVM Disk**


### `certificate.yml`


- **Download CA certificate installation script**
- **Check if downloaded file exists and has content**
- **Check first few lines of downloaded file for debugging**
- **Display download result for debugging**
- **Check if CA script download was successful**
- **Validate downloaded script is executable**
- **Display script validation result**
- **Execute CA certificate installation script**
- **Remove temporary installation script**
- **Display certificate installation output**


### `hostname.yml`


- **Set hostname on the node**


### `hosts.yml`


- **Gather network facts**
- **Check if /etc/hosts is accessible**
- **Update hostname entries in /etc/hosts**


### `epel/RedHat.yml`


- **Setup EPEL repo**


### `patches/security/apt-sources.yml`


- **Update package cache (apt)**
- **Create security sources directory**
- **Create security-only sources list for Debian**
- **Create security-only sources list for Ubuntu**
- **Update package cache with security sources**
- **Apply security updates for Debian systems**
- **Apply security updates for Ubuntu systems**
- **Remove unused packages after security updates**
- **Display security update results**


### `patches/security/apt-unattended.yml`


- **Update package cache (apt)**
- **Install required packages for security updates**
- **Configure unattended-upgrades for security updates only**
- **Configure unattended-upgrades automatic updates**
- **Enable unattended-upgrades service**
- **Run dry-run to check available security updates**
- **Check if security updates are available**
- **Display available security updates**
- **Display no security updates message**
- **Apply security updates using unattended-upgrades**
- **Clean up package cache after security updates**


### `patches/security/apt.yml`


- **Apply security updates using unattended-upgrades method**
- **Apply security updates using apt-sources method**


### `patches/security/yum.yml`


- **Update package cache (yum)**
- **Apply security updates only (yum)**


### `patches/security/dnf.yml`


- **Update package cache (dnf)**
- **Apply security updates only (dnf)**


### `patches/os/apt.yml`


- **Update package cache (apt)**
- **Get list of upgradable packages excluding kernel packages**
- **Apply OS updates excluding kernel packages (apt)**


### `patches/os/yum.yml`


- **Update package cache (yum)**
- **Apply all available OS updates excluding kernel (yum)**


### `patches/os/dnf.yml`


- **Update package cache (dnf)**
- **Apply all available OS updates excluding kernel (dnf)**


### `patches/kernel/apt.yml`


- **Update package cache (apt)**
- **Apply kernel updates (apt)**


### `patches/kernel/yum.yml`


- **Apply kernel updates (yum)**


### `patches/kernel/dnf.yml`


- **Apply kernel updates (dnf)**


### `package/apt.yml`


- **Install prerequisites (Debian family)**


### `package/yum.yml`


- **Install prerequisites (RedHat family with yum)**


### `package/dnf.yml`


- **Install prerequisites (RedHat family with dnf)**




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
