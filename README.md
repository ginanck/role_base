# Ansible Role: Base System Configuration

A comprehensive Ansible role for base system configuration that handles essential server setup tasks including package management, patch management, security configuration, network settings, time synchronization, and storage management. This role provides a solid foundation for server provisioning across multiple Linux distributions.

## Requirements

- **Ansible Version**: 2.9 or higher
- **Supported Operating Systems**:
  - Ubuntu 18.04, 20.04, 22.04, 24.04
  - Debian 10, 11, 12
  - CentOS/RHEL 7, 8, 9
  - Rocky Linux 8, 9
  - AlmaLinux 8, 9

## Role Variables

### Core System Configuration

- `base_domain`: (string, default: "internal.guru") Base domain name for hostname resolution
- `base_hostname`: (string, default: "") System hostname to configure
- `base_hostname_configured`: (bool, default: true) Whether to configure the system hostname
- `base_timezone`: (string, default: "Europe/Helsinki") System timezone configuration

### Certificate Management

- `base_ca_install_enabled`: (bool, default: true) Enable installation of CA certificates
- `base_ca_script_url`: (string, default: "http://ca.internal.guru/scripts/install-linux.sh") URL for CA certificate installation script

### Cloud and System Configuration

- `base_configure_cloud_init`: (bool, default: true) Configure cloud-init settings
- `base_swap_disabled`: (bool, default: false) Whether to disable swap

### Package Management

- `base_default_packages`: (list, default: ["vim", "net-tools", "tar", "unzip", "gzip", "telnet", "chrony", "wget", "curl", "llvm", "lvm2", "git"]) Default packages to install on all systems
- `base_additional_packages`: (list, default: []) Additional packages to install beyond defaults
- `base_os_specific_packages`: (list, varies by OS) OS-specific packages automatically included based on distribution
- `base_pyenv_build_dependencies`: (list, varies by OS) Python environment build dependencies

### OS Patch Management

- `base_apply_os_patches`: (bool, default: true) Apply operating system patches
- `base_apply_kernel_patches`: (bool, default: true) Apply kernel patches
- `base_apply_security_patches`: (bool, default: true) Apply security patches
- `base_reboot_after_patches`: (bool, default: false) Automatically reboot after applying patches
- `base_reboot_timeout`: (int, default: 600) Timeout in seconds for reboot operations
- `base_disable_gpg_check`: (bool, default: true) Disable GPG signature checking for package operations

### Security Configuration (Debian/Ubuntu)

- `base_security_method`: (string, default: "unattended-upgrades") Security update method ("unattended-upgrades" or "apt-sources")
- `base_security_auto_reboot`: (bool, default: false) Enable automatic reboot for security updates
- `base_security_auto_reboot_time`: (string, default: "02:00") Time for automatic security reboots
- `base_security_remove_unused_deps`: (bool, default: true) Remove unused dependencies during security updates
- `base_security_auto_updates_daily`: (bool, default: false) Enable daily automatic security updates

### Network Configuration

- `base_hostname_entries`: (list, default: []) Additional hostname entries for /etc/hosts
- `base_resolv_conf_managed`: (bool, default: true) Manage /etc/resolv.conf configuration
- `base_resolv_nameserver_entries`: (list, default: ["172.16.2.21"]) DNS nameserver entries
- `base_resolv_nameserver_search_domains`: (list, default: ["."]) DNS search domains
- `base_resolv_nameserver_resolv_options`: (list, default: ["edns0", "trust-ad"]) DNS resolver options

### Time Synchronization (Chrony)

- `base_chrony_keys`: (list, default: []) Chrony authentication keys
- `base_chrony_config`: (dict) Comprehensive chrony configuration with the following structure:
  - `server`: NTP server configuration with name list and parameters
  - `sourcedir`: Source directory for additional configuration
  - `driftfile`: Path to drift file
  - `makestep`: Step threshold and limit
  - `rtcsync`: Enable RTC synchronization
  - `hwtimestamp`: Hardware timestamping configuration
  - `minsources`: Minimum number of sources
  - `allow`: Network access control (optional)
  - `local`: Local clock configuration
  - `authselectmode`: Authentication mode
  - `keyfile`: Key file path
  - `ntsdumpdir`: NTS dump directory
  - `leapsecmode`: Leap second handling mode
  - `leapsectz`: Leap second timezone
  - `logdir`: Log directory
  - `log`: Logging configuration

### Storage Management (LVM)

- `base_lvm_disks`: (list, default: []) LVM disk configuration with the following structure:
  ```yaml
  - pv: /dev/vdb                    # Physical volume device
    vg: data                        # Volume group name
    lv:                            # Logical volumes list
      - name: jenkins              # Logical volume name
        size: 15G                  # Size specification
        path: /data/jenkins        # Mount point
  ```

### Platform-Specific Variables

**RedHat-based systems** (automatically included from `vars/RedHat.yml`):
- `base_os_specific_packages`: ["policycoreutils-python-utils", "python3-libselinux", "python3-policycoreutils", "bind-utils", "epel-release", "lvm2-libs"]
- `base_pyenv_build_dependencies`: ["make", "gcc", "patch", "zlib-devel", "bzip2", "bzip2-devel", "readline-devel", "sqlite", "sqlite-devel", "openssl-devel", "tk-devel", "libffi-devel", "xz-devel", "libuuid-devel", "gdbm-libs", "libnsl2"]

**Debian-based systems** (automatically included from `vars/Debian.yml`):
- `base_os_specific_packages`: ["python3-selinux", "selinux-utils", "policycoreutils", "bind9-utils", "liblvm2-dev"]
- `base_pyenv_build_dependencies`: ["make", "build-essential", "libssl-dev", "zlib1g-dev", "libbz2-dev", "libreadline-dev", "libsqlite3-dev", "libncursesw5-dev", "xz-utils", "tk-dev", "libxml2-dev", "libxmlsec1-dev", "libffi-dev", "liblzma-dev"]

## Dependencies

This role has no external dependencies on other Ansible roles.

## Example Playbooks

### Basic Usage

```yaml
---
- hosts: servers
  become: yes
  roles:
    - role: role_base
      vars:
        base_hostname: "web01"
        base_domain: "example.com"
        base_timezone: "UTC"
        base_additional_packages:
          - htop
          - iotop
```

### Security-Focused Configuration

```yaml
---
- hosts: production
  become: yes
  roles:
    - role: role_base
      vars:
        # Security patch configuration
        base_apply_security_patches: true
        base_apply_os_patches: true
        base_apply_kernel_patches: true
        base_reboot_after_patches: true
        base_reboot_timeout: 300
        
        # Debian/Ubuntu security automation
        base_security_method: "unattended-upgrades"
        base_security_auto_reboot: true
        base_security_auto_reboot_time: "03:00"
        base_security_remove_unused_deps: true
        base_security_auto_updates_daily: true
        
        # CA certificate installation
        base_ca_install_enabled: true
        base_ca_script_url: "https://ca.company.com/install.sh"
```

### Network and DNS Configuration

```yaml
---
- hosts: infrastructure
  become: yes
  roles:
    - role: role_base
      vars:
        # Hostname and domain setup
        base_hostname: "infra01"
        base_domain: "internal.company.com"
        base_hostname_configured: true
        
        # DNS configuration
        base_resolv_conf_managed: true
        base_resolv_nameserver_entries:
          - "10.0.1.10"
          - "10.0.1.11"
        base_resolv_nameserver_search_domains:
          - "company.com"
          - "internal.company.com"
        base_resolv_nameserver_resolv_options:
          - "edns0"
          - "trust-ad"
          - "ndots:2"
        
        # Additional hosts entries
        base_hostname_entries:
          - ip: "10.0.1.100"
            hostname: "database"
            fqdn: "database.internal.company.com"
          - ip: "10.0.1.101"
            hostname: "cache"
            fqdn: "cache.internal.company.com"
```

### Time Synchronization Configuration

```yaml
---
- hosts: all
  become: yes
  roles:
    - role: role_base
      vars:
        base_timezone: "America/New_York"
        base_chrony_config:
          server:
            param: "iburst maxpoll 6"
            name:
              - "time1.company.com"
              - "time2.company.com"
              - "pool.ntp.org"
          sourcedir: "/etc/chrony/sources.d"
          driftfile: "/var/lib/chrony/drift"
          makestep: "1.0 3"
          rtcsync: yes
          hwtimestamp: "*"
          minsources: 2
          allow:
            - "192.168.0.0/16"
            - "10.0.0.0/8"
          local:
            stratum: 10
          authselectmode: "require"
          keyfile: "/etc/chrony.keys"
          logdir: "/var/log/chrony"
          log:
            measurements: yes
            statistics: yes
            tracking: yes
        base_chrony_keys:
          - "1 SHA256 HEX:1234567890ABCDEF"
```

### Storage Management with LVM

```yaml
---
- hosts: database_servers
  become: yes
  roles:
    - role: role_base
      vars:
        # Disable swap for database performance
        base_swap_disabled: true
        
        # LVM configuration for data storage
        base_lvm_disks:
          - pv: "/dev/vdb"
            vg: "data"
            lv:
              - name: "mysql"
                size: "50G"
                path: "/var/lib/mysql"
              - name: "logs"
                size: "20G"
                path: "/var/log/mysql"
          
          - pv: "/dev/vdc"
            vg: "backup"
            lv:
              - name: "backup_storage"
                size: "100%FREE"
                path: "/backup"
```

### Development Environment Setup

```yaml
---
- hosts: development
  become: yes
  roles:
    - role: role_base
      vars:
        # Minimal patching for development
        base_apply_security_patches: false
        base_apply_os_patches: false
        base_apply_kernel_patches: false
        base_reboot_after_patches: false
        
        # Development packages (includes Python build dependencies)
        base_additional_packages:
          - docker.io
          - docker-compose
          - nodejs
          - npm
          - python3-pip
          - python3-venv
        
        # Relaxed security for development
        base_disable_gpg_check: true
        base_security_auto_updates_daily: false
        
        # Configure cloud-init for development VMs
        base_configure_cloud_init: true
```

### Enterprise Compliance Configuration

```yaml
---
- hosts: compliance_servers
  become: yes
  roles:
    - role: role_base
      vars:
        # Strict patch management
        base_apply_security_patches: true
        base_apply_os_patches: true
        base_apply_kernel_patches: true
        base_reboot_after_patches: true
        base_disable_gpg_check: false
        
        # Enterprise DNS
        base_resolv_nameserver_entries:
          - "10.1.1.10"
          - "10.1.1.11"
        base_resolv_nameserver_search_domains:
          - "corp.enterprise.com"
        
        # Cloud-init disabled for compliance
        base_configure_cloud_init: false
        
        # Comprehensive package set
        base_additional_packages:
          - aide
          - rkhunter
          - chkrootkit
          - auditd
          - rsyslog
          - logrotate
        
        # Enterprise time sources
        base_chrony_config:
          server:
            param: "iburst minpoll 4 maxpoll 6"
            name:
              - "ntp1.enterprise.com"
              - "ntp2.enterprise.com"
          authselectmode: "require"
          minsources: 2
```

### Patch Management Only (Minimal Configuration)

```yaml
---
- hosts: existing_servers
  become: yes
  roles:
    - role: role_base
      vars:
        # Only apply patches, don't configure other aspects
        base_hostname_configured: false
        base_configure_cloud_init: false
        base_resolv_conf_managed: false
        base_ca_install_enabled: false
        
        # Focus on security patches only
        base_apply_security_patches: true
        base_apply_os_patches: false
        base_apply_kernel_patches: false
        base_reboot_after_patches: false
        
        # Don't install additional packages
        base_additional_packages: []
```

## Detailed Feature Sections

### Package Management

The role provides comprehensive package management across different Linux distributions:

- **Universal Packages**: A curated set of essential packages installed on all systems including system utilities, network tools, version control, and management tools
- **OS-Specific Packages**: Automatically includes distribution-specific packages for optimal functionality (SELinux tools, DNS utilities, LVM libraries)
- **Python Build Dependencies**: Complete development toolchain for Python environments when needed
- **Custom Package Lists**: Easily extend with additional packages for specific use cases

**Platform-Specific Behavior**:
- **RedHat/CentOS**: Includes EPEL repository and SELinux tools, GCC toolchain
- **Debian/Ubuntu**: Includes security repository configuration and build-essential
- **All Platforms**: LVM tools, network utilities, and system administration packages

### Patch Management System

Advanced patch management with granular control over different types of updates:

**Security Patches**: Priority handling of security updates with automated installation options. When enabled, takes precedence over other patch types to ensure security-first approach.

**Kernel Patches**: Separate control for kernel updates with intelligent reboot coordination. Automatically detects when reboots are required and can perform them safely.

**OS Patches**: General system updates with flexible scheduling. Includes all non-security package updates for maintaining system currency.

**Reboot Management**: Intelligent reboot handling with configurable timeouts and conditional execution based on patch types applied.

**Debian/Ubuntu Specific Features**:
- **Unattended Upgrades**: Fully automated security update installation with configurable scheduling
- **APT Sources Method**: Manual security repository configuration for custom update sources
- **Dependency Cleanup**: Automatic removal of unused packages to maintain system cleanliness
- **Scheduled Reboots**: Time-based automatic reboots for security updates with customizable maintenance windows

### Network Configuration

Comprehensive network setup including DNS, hostname resolution, and service discovery:

**DNS Management**: Complete /etc/resolv.conf configuration supporting multiple nameservers, custom search domains, and advanced resolver options like EDNS and DNSSEC.

**Hostname Resolution**: Automated /etc/hosts management with support for custom entries, FQDN resolution, and integration with dynamic infrastructure.

**FQDN Setup**: Proper hostname and domain configuration ensuring consistent network identity across all managed systems.

**Service Discovery**: Custom hostname entries support for internal service discovery without requiring external DNS infrastructure.

### Time Synchronization

Enterprise-grade time synchronization using Chrony with comprehensive configuration options:

**NTP Sources**: Multiple time server configuration with fallback options, custom polling intervals, and server-specific parameters.

**Network Access**: Configurable client access controls allowing other systems to synchronize time through managed servers.

**Authentication**: Key-based authentication support for secure time synchronization in enterprise environments.

**Hardware Integration**: Hardware timestamping support for high-precision timing requirements in financial and scientific applications.

**Logging and Monitoring**: Comprehensive logging of time synchronization events, drift measurements, and source reliability statistics.

### Storage Management

Flexible LVM-based storage management supporting complex storage layouts:

**Physical Volume Management**: Automatic physical volume creation and configuration with support for multiple storage devices.

**Volume Group Setup**: Logical organization of storage resources allowing for flexible capacity management and future expansion.

**Logical Volume Creation**: Automated logical volume creation with custom sizing options including fixed sizes and percentage-based allocation.

**Mount Point Management**: Automatic filesystem creation and mounting with proper ownership and permission settings.

**Growth Planning**: Support for percentage-based sizing (100%FREE) enabling efficient use of available storage and simplified future expansion.

### Security and Certificate Management

Comprehensive security baseline configuration:

**CA Certificate Installation**: Automated installation of organization-specific certificate authorities enabling secure internal communications.

**Swap Management**: Optional swap disabling for security-sensitive applications and performance optimization.

**GPG Verification**: Configurable package signature verification with flexibility for development and testing environments.

**Update Automation**: Intelligent security update automation balancing security requirements with operational stability.

## Best Practices

### Security Considerations

- Enable `base_apply_security_patches: true` in all production environments to maintain security posture
- Use `base_security_auto_reboot: true` carefully in clustered environments - consider maintenance windows and load balancing
- Configure `base_ca_install_enabled: true` only with trusted certificate sources from your organization
- Set `base_disable_gpg_check: false` in high-security environments to ensure package authenticity
- Consider using `base_security_method: "apt-sources"` for environments requiring custom security repositories

### Performance Optimization

- Set `base_swap_disabled: true` for database servers and memory-intensive applications to prevent performance degradation
- Use `base_chrony_config.minsources: 3` or higher for critical time-sensitive applications requiring high availability
- Configure appropriate `base_reboot_timeout` values based on your infrastructure boot times and service dependencies
- Consider `base_apply_kernel_patches: false` for systems where uptime is critical and kernel updates can be scheduled separately

### Network Planning

- Design `base_resolv_nameserver_search_domains` carefully to minimize DNS query overhead and avoid unnecessary lookups
- Use `base_hostname_entries` for critical service discovery to reduce external DNS dependencies
- Plan `base_domain` naming convention consistently across your organization for proper certificate management
- Consider DNS caching and resolver performance when configuring `base_resolv_nameserver_resolv_options`

### Storage Planning

- Plan LVM layouts in `base_lvm_disks` with future growth in mind, using volume groups strategically
- Use percentage sizing (`100%FREE`) for flexible capacity allocation in primary data volumes
- Consider backup strategies when designing volume layouts - separate volume groups for data and backup storage
- Plan logical volume names and mount points according to application requirements and operational procedures

### Operational Considerations

- Test patch management settings in development environments before applying to production
- Document custom `base_additional_packages` requirements for environment reproducibility
- Monitor `base_chrony_config` settings for time synchronization health in distributed systems
- Plan maintenance windows around `base_reboot_after_patches` schedules

## Troubleshooting

### Common Issues

**Patch Installation Failures**: 
- Check `base_disable_gpg_check` setting - may need to be `true` for repositories with signing issues
- Verify repository accessibility and network connectivity
- Review package manager logs for specific error messages
- Consider temporary network or repository issues

**Time Synchronization Problems**: 
- Verify NTP server accessibility from managed hosts
- Check firewall rules for NTP traffic (UDP port 123)
- Review `base_chrony_config.server` configuration for correct server addresses
- Monitor chronyd logs for synchronization status

**Storage Configuration Issues**: 
- Ensure block devices specified in `base_lvm_disks` actually exist on target systems
- Verify sufficient disk space for requested logical volume sizes
- Check for existing LVM configurations that might conflict
- Review /var/log/messages for storage-related errors

**Hostname Resolution Problems**: 
- Verify DNS configuration in `base_resolv_nameserver_entries`
- Check network connectivity to configured DNS servers
- Review `/etc/hosts` for conflicts with `base_hostname_entries`
- Test DNS resolution manually with tools like `dig` or `nslookup`

**Package Installation Issues**:
- Review `base_os_specific_packages` compatibility with target OS versions
- Check repository availability for `base_additional_packages`
- Verify package name differences between distributions
- Consider dependency conflicts with existing packages

### Debug Mode

Enable Ansible verbose logging with `-vvv` flag to get detailed information about task execution:

```bash
ansible-playbook -i inventory playbook.yml -vvv
```

For role-specific debugging, add debug tasks to verify variable values:

```yaml
- debug:
    var: base_chrony_config
  when: base_chrony_config is defined
```

### Log Files

Monitor these log files for troubleshooting:

- **Patch Management**: `/var/log/apt/` (Debian/Ubuntu), `/var/log/yum.log` (RHEL/CentOS)
- **Time Synchronization**: `/var/log/chrony/` (if configured)
- **System Events**: `/var/log/messages` or `/var/log/syslog`
- **Package Management**: Distribution-specific package manager logs

## License

MIT

## Author Information

This role has been created by gkorkmaz  
GitHub: https://github.com/ginanck