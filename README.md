# Ansible Role: role_base

A comprehensive base role for Linux systems that provides essential system configuration, package management, OS patching, security updates, and infrastructure setup. This role is designed to establish a secure, standardized baseline for servers across different Linux distributions.

## Features

- **Multi-Platform Support**: Ubuntu, Debian, RHEL, CentOS, Rocky Linux
- **Comprehensive Patch Management**: Security-only, OS, and kernel patches with granular control
- **System Configuration**: Hostname, timezone, hosts file, DNS resolver management
- **Service Management**: Chrony NTP, firewall, SELinux, cloud-init configuration
- **Storage Management**: LVM disk setup and partition management
- **Security Hardening**: CA certificate installation, swap disabling, security patches
- **Package Management**: Default and additional package installation with OS-specific packages

## Requirements

- **Ansible Version**: 2.9+
- **Supported Operating Systems**:
  - Ubuntu 18.04+ (Bionic, Focal, Jammy, Noble)
  - Debian 9+ (Stretch, Buster, Bullseye, Bookworm)
  - RHEL/CentOS 7+
  - Rocky Linux 8+
  - AlmaLinux 8+

## Dependencies

This role has no external role dependencies. It uses the following Ansible collections:
- `community.general` (version 6.6.1)
- `ansible.posix` (version 1.5.4)

## Role Variables

### Core System Configuration

- `base_hostname`: (string, default: "") Hostname to set for the system
- `base_hostname_configured`: (bool, default: true) Whether to configure the hostname
- `base_domain`: (string, default: "internal.guru") Domain name for the system
- `base_timezone`: (string, default: "Europe/Helsinki") System timezone to configure

### OS Patch Management

- `base_apply_os_patches`: (bool, default: true) Whether to apply OS-level patches/updates
- `base_apply_kernel_patches`: (bool, default: true) Whether to apply kernel patches (typically requires reboot)
- `base_apply_security_patches`: (bool, default: true) Whether to apply only security patches (mutually exclusive with other patch types)
- `base_reboot_after_patches`: (bool, default: false) Whether to automatically reboot system after patches if required
- `base_reboot_timeout`: (int, default: 600) Timeout in seconds for reboot operation
- `base_disable_gpg_check`: (bool, default: true) Disable GPG signature checking for packages (useful for containers/testing)

### Security Update Configuration (Debian/Ubuntu)

- `base_security_method`: (string, default: "unattended-upgrades") Method for security updates ("unattended-upgrades" or "apt-sources")
- `base_security_auto_reboot`: (bool, default: false) Allow automatic reboot after security updates when using unattended-upgrades
- `base_security_auto_reboot_time`: (string, default: "02:00") Time for automatic reboot if enabled (HH:MM format)
- `base_security_remove_unused_deps`: (bool, default: true) Remove unused dependencies after security updates
- `base_security_auto_updates_daily`: (bool, default: false) Enable daily automatic security updates via unattended-upgrades

### Package Management

- `base_default_packages`: (list, default: comprehensive list) List of default packages to install on all systems
- `base_additional_packages`: (list, default: []) Additional packages to install beyond defaults
- `base_os_specific_packages`: (list, platform-specific) OS-specific packages defined in vars files (RedHat.yml, Debian.yml)
- `base_pyenv_build_dependencies`: (list, platform-specific) Python build dependencies for development environments

### Network Configuration

- `base_resolv_conf_managed`: (bool, default: true) Whether to manage /etc/resolv.conf
- `base_resolv_nameserver_entries`: (list, default: ["172.16.2.21"]) DNS nameserver IP addresses
- `base_resolv_nameserver_search_domains`: (list, default: ["."]) DNS search domains
- `base_resolv_nameserver_resolv_options`: (list, default: ["edns0", "trust-ad"]) DNS resolver options

### Hosts File Management

- `base_hostname_entries`: (list, default: []) Additional hosts entries to add to /etc/hosts

### System Services Configuration

- `base_configure_cloud_init`: (bool, default: true) Whether to configure cloud-init to preserve hostname
- `base_swap_disabled`: (bool, default: false) Whether to disable swap completely

### Security & Certificate Management

- `base_ca_install_enabled`: (bool, default: true) Whether to install custom CA certificates
- `base_ca_script_url`: (string, default: "http://ca.internal.guru/scripts/install-linux.sh") URL for CA certificate installation script

### Time Synchronization (Chrony)

- `base_chrony_config`: (dict) Complex chrony configuration object with multiple sub-keys
- `base_chrony_keys`: (list, default: []) Chrony authentication keys

### Storage Management (LVM)

- `base_lvm_disks`: (list, default: []) List of LVM disk configurations

### Platform-Specific Variables

**RedHat-based systems** (`vars/RedHat.yml`):
- `base_os_specific_packages`: Includes policycoreutils-python-utils, python3-libselinux, bind-utils, epel-release, etc.
- `base_pyenv_build_dependencies`: GCC, development libraries for Python building

**Debian-based systems** (`vars/Debian.yml`):
- `base_os_specific_packages`: Includes python3-selinux, selinux-utils, bind9-utils, etc.
- `base_pyenv_build_dependencies`: Build-essential, development libraries for Python building

## Example Playbooks

### Basic Usage - Standard Production Setup

```yaml
- hosts: production_servers
  become: true
  roles:
    - role: role_base
      # Standard OS patching without kernel updates to avoid unexpected reboots
      base_apply_os_patches: true
      base_apply_kernel_patches: false
      base_apply_security_patches: false
      base_reboot_after_patches: false
      
      # Set hostname and domain
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "company.local"
      
      # Add essential monitoring tools
      base_additional_packages:
        - htop
        - tree
        - curl
        - wget
        - jq
        - jq
```

### Security-Only Updates (Compliance Environments)

```yaml
- hosts: compliance_servers
  become: true
  roles:
    - role: role_base
      # Apply only security patches for compliance requirements
      base_apply_security_patches: true
      base_apply_os_patches: false      # Automatically disabled when security patches enabled
      base_apply_kernel_patches: false  # Automatically disabled when security patches enabled
      base_reboot_after_patches: false
      
      # Set hostname
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "compliance.local"
      
      # Minimal additional packages for compliance environments
      base_additional_packages:
        - aide          # File integrity monitoring
        - rkhunter      # Rootkit detection
        - chkrootkit    # Additional rootkit detection
```

### Advanced Ubuntu/Debian Security with Unattended-Upgrades

```yaml
- hosts: ubuntu_servers
  become: true
  roles:
    - role: role_base
      # Enable security patches with automated management
      base_apply_security_patches: true
      base_security_method: "unattended-upgrades"  # Official recommendation
      
      # Configure automatic security updates
      base_security_auto_updates_daily: true       # Enable daily automatic security updates
      base_security_auto_reboot: true              # Allow automatic reboot for security updates
      base_security_auto_reboot_time: "03:00"      # Schedule reboot at 3 AM
      base_security_remove_unused_deps: true       # Clean up unused packages
      
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "prod.example.com"
      base_timezone: "America/New_York"
```

### Alternative Ubuntu/Debian Security with Direct APT Sources

```yaml
- hosts: ubuntu_development
  become: true
  roles:
    - role: role_base
      # Use direct apt sources method for more control
      base_apply_security_patches: true
      base_security_method: "apt-sources"          # Direct apt module approach
      base_security_remove_unused_deps: true
      
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "dev.example.com"
      
      # Development packages
      base_additional_packages:
        - build-essential
        - nodejs
        - npm
        - docker.io
        - git-lfs
```

### Database Servers (No Kernel Updates)

```yaml
- hosts: database_servers
  become: true
  roles:
    - role: role_base
      # Apply OS patches but skip kernel to avoid unplanned reboots
      base_apply_os_patches: true
      base_apply_kernel_patches: false  # Critical: Skip kernel updates
      base_apply_security_patches: false
      base_reboot_after_patches: false
      
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "db.internal"
      
      # Database-specific packages
      base_additional_packages:
        - htop
        - iotop
        - sysstat
        - tcpdump
        - strace
```

### Development Environment with All Features

```yaml
- hosts: development_servers
  become: true
  roles:
    - role: role_base
      # Full patching including kernel updates (acceptable for dev)
      base_apply_os_patches: true
      base_apply_kernel_patches: true
      base_apply_security_patches: false
      base_reboot_after_patches: true   # Auto-reboot acceptable in dev
      base_reboot_timeout: 300          # Shorter timeout for dev
      
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "dev.internal"
      base_timezone: "UTC"
      
      # Development packages
      base_additional_packages:
        - vim-enhanced
        - tmux
        - screen
        - git
        - curl
        - wget
        - jq
        - tree
        - htop
        - iotop
        - strace
        - tcpdump
        - nmap
        - telnet
        - nc
```

### Complete Network and DNS Configuration

```yaml
- hosts: infrastructure_servers
  become: true
  roles:
    - role: role_base
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "infrastructure.local"
      base_timezone: "America/Chicago"
      
      # DNS configuration
      base_resolv_conf_managed: true
      base_resolv_nameserver_entries:
        - "8.8.8.8"
        - "8.8.4.4"
        - "1.1.1.1"
      base_resolv_nameserver_search_domains:
        - "infrastructure.local"
        - "local"
      base_resolv_nameserver_resolv_options:
        - "rotate"
        - "timeout:2"
        - "attempts:3"
      
      # Hosts file entries
      base_hostname_entries:
        - ip: "10.0.1.10"
          hostname: "db01"
          fqdn: "db01.infrastructure.local"
        - ip: "10.0.1.11"
          hostname: "web01"
          fqdn: "web01.infrastructure.local"
        - ip: "10.0.1.12"
          hostname: "api01"
          fqdn: "api01.infrastructure.local"
```

### Custom CA Certificate Installation

```yaml
- hosts: secure_servers
  become: true
  roles:
    - role: role_base
      # Security configuration
      base_ca_install_enabled: true
      base_ca_script_url: "https://ca.company.com/scripts/install-linux.sh"
      
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "secure.company.com"
      
      # Disable swap for security
      base_swap_disabled: true
```

### Advanced Chrony NTP Configuration

```yaml
- hosts: time_servers
  become: true
  roles:
    - role: role_base
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "time.internal"
      base_timezone: "UTC"
      
      # Advanced Chrony configuration
      base_chrony_config:
        server:
          param: "iburst prefer"
          name:
            - "pool.ntp.org"
            - "time.cloudflare.com"
            - "time.google.com"
        sourcedir: "/run/chrony-dhcp"
        driftfile: "/var/lib/chrony/drift"
        makestep: "1.0 3"
        rtcsync: yes
        hwtimestamp: "*"
        minsources: 3
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
          rtc: yes
```

### LVM Storage Configuration

```yaml
- hosts: storage_servers
  become: true
  roles:
    - role: role_base
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "storage.internal"
      
      # LVM disk configuration
      base_lvm_disks:
        - pv: "/dev/sdb"
          vg: "data"
          lv:
            - name: "web_data"
              size: "50G"
              path: "/var/www"
            - name: "app_logs"
              size: "20G"
              path: "/var/log/apps"
            - name: "backups"
              size: "100%FREE"
              path: "/backups"
        
        - pv: "/dev/sdc"
          vg: "database"
          lv:
            - name: "mysql_data"
              size: "80G"
              path: "/var/lib/mysql"
            - name: "mysql_logs"
              size: "20G"
              path: "/var/log/mysql"
```

### Cloud-Init and Container-Optimized Configuration

```yaml
- hosts: cloud_instances
  become: true
  roles:
    - role: role_base
      # Cloud-specific configuration
      base_configure_cloud_init: true
      base_swap_disabled: true          # Recommended for containers/cloud
      
      # Disable GPG checks for cloud environments
      base_disable_gpg_check: true
      
      # Minimal patching approach for cloud instances
      base_apply_os_patches: true
      base_apply_kernel_patches: false
      base_reboot_after_patches: false
      
      # System configuration
      base_hostname: "{{ inventory_hostname }}"
      base_domain: "cloud.company.com"
      
      # Cloud-optimized packages
      base_additional_packages:
        - cloud-init
        - cloud-utils
        - curl
        - wget
        - jq
```

### Complete Feature Demonstration

```yaml
- hosts: demo_servers
  become: true
  roles:
    - role: role_base
      # Patch management - all options demonstrated
      base_apply_os_patches: true
      base_apply_kernel_patches: true
      base_apply_security_patches: false
      base_reboot_after_patches: false
      base_reboot_timeout: 900
      base_disable_gpg_check: false
      
      # Security configuration for Ubuntu/Debian
      base_security_method: "unattended-upgrades"
      base_security_auto_reboot: false
      base_security_auto_reboot_time: "04:00"
      base_security_remove_unused_deps: true
      base_security_auto_updates_daily: false
      
      # System configuration
      base_hostname: "demo-{{ inventory_hostname }}"
      base_hostname_configured: true
      base_domain: "demo.example.com"
      base_timezone: "Europe/London"
      
      # Network configuration
      base_resolv_conf_managed: true
      base_resolv_nameserver_entries:
        - "192.168.1.1"
        - "8.8.8.8"
      base_resolv_nameserver_search_domains:
        - "demo.example.com"
        - "example.com"
      base_resolv_nameserver_resolv_options:
        - "edns0"
        - "trust-ad"
        - "rotate"
      
      # Hosts entries
      base_hostname_entries:
        - ip: "192.168.1.100"
          hostname: "demo-db"
          fqdn: "demo-db.demo.example.com"
        - ip: "192.168.1.101"
          hostname: "demo-web"
          fqdn: "demo-web.demo.example.com"
      
      # Services configuration
      base_configure_cloud_init: true
      base_swap_disabled: false
      
      # Security
      base_ca_install_enabled: true
      base_ca_script_url: "https://pki.example.com/install-ca.sh"
      
      # Package management
      base_additional_packages:
        - htop
        - tree
        - curl
        - wget
        - jq
        - vim
        - git
        - tmux
        - screen
        - net-tools
        - bind-utils
        - sysstat
        - iotop
        - tcpdump
        - strace
        - lsof
        - nc
        - telnet
        - rsync
        - unzip
        - zip
      
      # Chrony configuration
      base_chrony_config:
        server:
          param: "iburst"
          name:
            - "0.pool.ntp.org"
            - "1.pool.ntp.org"
            - "2.pool.ntp.org"
            - "3.pool.ntp.org"
        sourcedir: "/run/chrony-dhcp"
        driftfile: "/var/lib/chrony/drift"
        makestep: "1.0 3"
        rtcsync: yes
        hwtimestamp: "*"
        minsources: 2
        local:
          stratum: 10
        authselectmode: "require"
        keyfile: "/etc/chrony.keys"
        ntsdumpdir: "/var/lib/chrony"
        leapsecmode: "slew"
        leapsectz: "right/UTC"
        logdir: "/var/log/chrony"
        log:
          measurements: yes
          statistics: yes
          tracking: yes
      
      # LVM configuration
      base_lvm_disks:
        - pv: "/dev/sdb"
          vg: "app_data"
          lv:
            - name: "applications"
              size: "30G"
              path: "/opt/apps"
            - name: "logs"
              size: "20G"
              path: "/var/log/apps"
```

## Detailed Feature Documentation

### OS Patch Management

This role provides comprehensive patch management with clear separation between different types of updates:

#### Patch Categories

- **Security Patches** (`tasks/patches/security/`): Security-only updates using native OS tools
  - **RHEL/CentOS/Rocky**: Uses `yum/dnf` with `security: true` parameter
  - **Debian/Ubuntu**: Two methods available:
    - `unattended-upgrades` (default): Official Ubuntu/Debian recommendation with rich configuration
    - `apt-sources`: Direct apt module with security-only source lists for more control

- **OS Patches** (`tasks/patches/os/`): Updates for applications, libraries, and system packages (excluding kernel)
- **Kernel Patches** (`tasks/patches/kernel/`): Updates specifically for kernel packages

#### Execution Flow

1. **Security Patches**: Applied first if enabled (mutually exclusive with other patch types)
2. **OS Patches**: Applied if security patches are not enabled, excludes kernel packages
3. **Kernel Patches**: Applied separately if enabled and security patches are not enabled
4. **Reboot Detection**: Intelligently determines if a reboot is required
5. **Optional Reboot**: Performs reboot if configured and required

#### Platform-Specific Behavior

**Debian/Ubuntu (apt)**:

- Security updates support two methods:
  - `unattended-upgrades`: Official recommendation, includes automated scheduling, rich configuration, and conflict handling
  - `apt-sources`: Direct control using security-only repository sources for simpler, one-time updates
- OS patches exclude kernel packages using intelligent package filtering
- Kernel patches target `linux-image-generic`, `linux-headers-generic`, etc.

**RHEL/CentOS/Rocky (yum/dnf)**:

- Security updates use native `security: true` parameter
- OS patches use `exclude: kernel*` parameter to skip kernel packages
- Kernel patches update all `kernel*` packages
- Supports both yum (legacy) and dnf (modern) package managers

### System Configuration Features

#### Hostname and Domain Management

- Sets system hostname using Ansible's hostname module
- Updates `/etc/hosts` with proper FQDN resolution
- Configures cloud-init to preserve hostname settings
- Supports additional hosts file entries for infrastructure integration

#### Time Synchronization

- Comprehensive chrony NTP configuration
- Supports multiple NTP servers with customizable parameters
- Configurable drift files, logging, and authentication
- Platform-appropriate default configurations

#### Network Configuration

- DNS resolver management through `/etc/resolv.conf`
- Support for multiple nameservers and search domains
- Customizable resolver options (edns0, trust-ad, rotate, etc.)
- Backup and restoration of existing configurations

### Storage Management

#### LVM Configuration

- Automated LVM setup for additional storage
- Supports multiple physical volumes and volume groups
- Automatic partition growth for cloud environments
- XFS filesystem creation and mounting
- Proper fstab entries for persistent mounts

### Security Features

#### Certificate Management

- Custom CA certificate installation via downloadable scripts
- Validation of downloaded scripts before execution
- Secure script handling with proper cleanup
- Support for enterprise PKI integration

#### System Hardening

- Optional swap disabling for security-sensitive environments
- SELinux and AppArmor management
- Firewall service management (firewalld, ufw)
- Cloud-init security configurations

### Package Management

#### Multi-Platform Package Support

- Unified package installation across apt, yum, and dnf
- OS-specific package lists for platform requirements
- Default packages for essential system tools
- Development dependency packages for Python environments
- Additional package installation with dependency resolution

## Variable Examples and Formats

### Complex Variable Structures

#### base_chrony_config Structure

```yaml
base_chrony_config:
  server:                    # NTP server configuration
    param: "iburst prefer"   # Server parameters
    name:                    # List of NTP servers
      - "pool.ntp.org"
      - "time.google.com"
  sourcedir: "/run/chrony-dhcp"     # Source directory for DHCP configs
  driftfile: "/var/lib/chrony/drift" # Drift file location
  makestep: "1.0 3"               # Step threshold and limit
  rtcsync: yes                    # RTC synchronization
  hwtimestamp: "*"               # Hardware timestamping
  minsources: 2                  # Minimum sources required
  allow:                         # Allow NTP clients (optional)
    - "192.168.0.0/24"
    - "10.0.0.0/8"
  local:                         # Local reference configuration
    stratum: 10
  authselectmode: "require"      # Authentication mode
  keyfile: "/etc/chrony.keys"    # Key file location
  logdir: "/var/log/chrony"      # Log directory
  log:                           # Logging options
    measurements: yes
    statistics: yes
    tracking: yes
```

#### base_lvm_disks Structure

```yaml
base_lvm_disks:
  - pv: "/dev/sdb"              # Physical volume device
    vg: "data"                  # Volume group name
    lv:                         # List of logical volumes
      - name: "applications"    # Logical volume name
        size: "50G"             # Size (supports G, M, %FREE, etc.)
        path: "/opt/apps"       # Mount point
      - name: "logs"
        size: "30G"
        path: "/var/log/apps"
      - name: "cache"
        size: "100%FREE"        # Use remaining space
        path: "/var/cache/apps"
```

#### base_hostname_entries Structure

```yaml
base_hostname_entries:
  - ip: "192.168.1.10"                    # IP address
    hostname: "web01"                     # Short hostname
    fqdn: "web01.production.local"        # Fully qualified domain name (optional)
  - ip: "192.168.1.11"
    hostname: "db01"
    # FQDN will be auto-generated as db01.{{ base_domain }}
```

### List Variables with Examples

#### base_default_packages (Complete List)

```yaml
base_default_packages:
  - vim                    # Text editor
  - net-tools             # Network utilities (ifconfig, netstat)
  - tar                   # Archive utility
  - unzip                 # ZIP file extraction
  - gzip                  # Compression utility
  - telnet                # Network testing tool
  - chrony                # NTP daemon
  - wget                  # HTTP/FTP download tool
  - curl                  # HTTP client and library
  - llvm                  # Compiler infrastructure
  - lvm2                  # Logical Volume Manager
  - git                   # Version control system
```

#### base_additional_packages Examples

```yaml
# Monitoring and troubleshooting packages
base_additional_packages:
  - htop                  # Interactive process viewer
  - iotop                 # I/O monitoring
  - sysstat               # System performance tools (sar, iostat)
  - tcpdump               # Network packet analyzer
  - strace                # System call tracer
  - lsof                  # List open files
  - nc                    # Netcat networking utility
  - nmap                  # Network scanner
  - tree                  # Directory tree display
  - jq                    # JSON processor

# Development packages
base_additional_packages:
  - build-essential       # Compilation tools (Debian/Ubuntu)
  - gcc                   # C compiler (RedHat/CentOS)
  - make                  # Build automation
  - git-lfs               # Git Large File Storage
  - nodejs                # JavaScript runtime
  - npm                   # Node.js package manager
  - python3-pip          # Python package installer
  - docker.io             # Container runtime
  - tmux                  # Terminal multiplexer
  - screen                # Terminal session manager

# Security tools
base_additional_packages:
  - aide                  # Advanced Intrusion Detection Environment
  - rkhunter             # Rootkit Hunter
  - chkrootkit           # Check Rootkit
  - clamav               # Antivirus engine
  - fail2ban             # Intrusion prevention
```

## Use Cases and Best Practices

### Production Server Recommendations

1. **Disable kernel patching** in production to control reboot timing
2. **Use security-only patches** for compliance environments
3. **Configure custom CA certificates** for enterprise environments
4. **Set appropriate DNS servers** for your infrastructure
5. **Use LVM configuration** for flexible storage management

### Development Environment Optimizations

1. **Enable automatic reboots** since downtime is acceptable
2. **Install comprehensive development packages**
3. **Use shorter reboot timeouts** for faster iterations
4. **Enable all patch types** for up-to-date development environment

### Security Considerations

1. **Review CA certificate scripts** before deployment
2. **Use unattended-upgrades method** for Ubuntu/Debian in production
3. **Configure swap disabling** for security-sensitive workloads
4. **Set appropriate file permissions** for chrony keys
5. **Use DNS security options** (edns0, trust-ad) when supported

### Performance Tuning

1. **Separate OS and kernel patches** to minimize reboot frequency
2. **Use package caching** to reduce network overhead
3. **Configure appropriate log retention** for chrony
4. **Use local NTP servers** when available for better accuracy

## Troubleshooting

### Common Issues

#### Patch Management

- **Issue**: Patches fail with GPG signature errors
  - **Solution**: Set `base_disable_gpg_check: true` temporarily or configure proper GPG keys
  - **Prevention**: Use proper repository configuration with valid GPG keys

- **Issue**: Unattended-upgrades not working on Ubuntu/Debian
  - **Solution**: Check `/var/log/unattended-upgrades/` logs and ensure proper configuration
  - **Alternative**: Switch to `base_security_method: "apt-sources"` for simpler approach

#### LVM Configuration

- **Issue**: LVM creation fails on cloud instances
  - **Solution**: Ensure partitions are grown first; check `base_lvm_disks` device paths
  - **Debug**: Use `lsblk` and `fdisk -l` to verify device availability

#### DNS Resolution

- **Issue**: DNS resolution not working after role execution
  - **Solution**: Verify `base_resolv_nameserver_entries` contains valid DNS servers
  - **Check**: Ensure DNS servers are reachable from target hosts

### Logging and Debugging

#### Enable Verbose Output

```yaml
- hosts: debug_servers
  become: true
  roles:
    - role: role_base
      # Enable verbose output for troubleshooting
  vars:
    ansible_verbosity: 2
```

#### Check Role Execution Status

```bash
# Check systemd services status
systemctl status chronyd
systemctl status unattended-upgrades

# Check patch status
apt list --upgradable                    # Debian/Ubuntu
yum check-update                         # RHEL/CentOS with yum
dnf check-update                         # RHEL/CentOS with dnf

# Check LVM status
lvs
vgs
pvs

# Check network configuration
systemd-resolve --status                 # Modern systems
cat /etc/resolv.conf                     # Traditional systems
```

## License

BSD-3-Clause

## Author Information

This role is maintained by gkorkmaz. For questions, issues, or contributions, please refer to the project repository.

For more detailed information about patch management structure and implementation, see [docs/patch_management.md](docs/patch_management.md).
