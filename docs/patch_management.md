# Patch Management Structure

This document explains the organization and logic behind the patch management system in this role.

## Directory Structure

```
tasks/patches/
├── security/               # Security-only patches
│   ├── apt.yml            # Debian/Ubuntu security patches
│   ├── dnf.yml            # Modern RHEL/Rocky/Fedora security patches
│   └── yum.yml            # Legacy RHEL/CentOS security patches
├── os/                     # OS-level patches (excluding kernel)
│   ├── apt.yml            # Debian/Ubuntu OS patches
│   ├── dnf.yml            # Modern RHEL/Rocky/Fedora OS patches
│   └── yum.yml            # Legacy RHEL/CentOS OS patches
└── kernel/                 # Kernel-specific patches
    ├── apt.yml            # Debian/Ubuntu kernel patches
    ├── dnf.yml            # Modern RHEL/Rocky/Fedora kernel patches
    └── yum.yml            # Legacy RHEL/CentOS kernel patches
```

## Patch Categories

### Security Patches (`patches/security/`)
- **Purpose**: Apply only security-related updates
- **Priority**: Highest - applied first if enabled
- **Mutually Exclusive**: When enabled, OS and kernel patches are skipped
- **RHEL Support**: Native `security: true` parameter
- **Debian Support**: Two methods available:
  1. **unattended-upgrades** (default): Official recommendation, rich features
  2. **apt-sources**: Direct control with specific security source lists

#### Debian/Ubuntu Security Methods Comparison

| Feature | unattended-upgrades | apt-sources |
|---------|-------------------|-------------|
| **Official Recommendation** | ✅ Yes | ❌ No |
| **Automatic Scheduling** | ✅ Built-in | ❌ Manual only |
| **Rich Configuration** | ✅ Extensive | ❌ Limited |
| **Conflict Handling** | ✅ Advanced | ❌ Basic |
| **Performance** | ❌ Slower | ✅ Faster |
| **Simplicity** | ❌ Complex setup | ✅ Simple |
| **Control** | ❌ Less direct | ✅ More direct |

**Recommendation**: Use `unattended-upgrades` for production systems requiring automated security updates, and `apt-sources` for simple, one-time security updates or when you need direct control.

### OS Patches (`patches/os/`)
- **Purpose**: Update applications, libraries, and system packages
- **Excludes**: Kernel packages to prevent unnecessary reboots
- **When Applied**: Always when `base_apply_os_patches` is `true`
- **Reboot Required**: Usually no, unless certain system services are updated

### Kernel Patches (`patches/kernel/`)
- **Purpose**: Update kernel and kernel-related packages
- **Includes**: `kernel*` packages
- **When Applied**: Only when `base_apply_kernel_patches` is `true`
- **Reboot Required**: Almost always requires reboot

## Execution Flow

1. **Security Patches**: Applied first if enabled (mutually exclusive with other types)
2. **OS Patches**: Applied if security patches are disabled
3. **Kernel Patches**: Applied if security patches are disabled and kernel patches are enabled
4. **Reboot Detection**: Checks if reboot is needed based on what was updated
5. **Optional Reboot**: Reboots if configured and required

## Configuration Options

| Variable | Default | Description |
|----------|---------|-------------|
| `base_apply_security_patches` | `false` | Apply only security patches |
| `base_apply_os_patches` | `true` | Apply OS-level patches |
| `base_apply_kernel_patches` | `true` | Apply kernel patches |
| `base_reboot_after_patches` | `false` | Auto-reboot if needed |

## Use Cases

### Security-Only Updates (Compliance)
```yaml
base_apply_security_patches: true
base_apply_os_patches: false      # Automatically disabled
base_apply_kernel_patches: false  # Automatically disabled
```

### Standard Production (Recommended)
```yaml
base_apply_security_patches: false
base_apply_os_patches: true
base_apply_kernel_patches: false  # Schedule kernel updates separately
base_reboot_after_patches: false
```

### Development Environment
```yaml
base_apply_security_patches: false
base_apply_os_patches: true
base_apply_kernel_patches: true
base_reboot_after_patches: true   # Auto-reboot is acceptable
```

### Security-Only Updates
```yaml
base_apply_os_patches: true
base_apply_kernel_patches: false
base_security_updates_only: true  # RHEL-based systems only
```

## Platform-Specific Behavior

### Debian/Ubuntu (apt)
- Uses `apt-mark hold/unhold` to exclude kernel packages during OS patching
- Kernel patches update `linux-image-generic`, `linux-headers-generic`, etc.

### RHEL/CentOS/Rocky (yum/dnf)
- Uses `exclude: kernel*` parameter to skip kernel packages during OS patching
- Kernel patches update all `kernel*` packages
- Supports security-only updates with `security: true` parameter

## Benefits of This Structure

1. **Clear Separation**: Easy to understand what each patch category does
2. **Maintainable**: Each file has a single, clear responsibility
3. **Flexible**: Can easily add new package managers or modify behavior
4. **Production-Safe**: Default behavior avoids unexpected reboots
5. **Extensible**: Easy to add new patch categories or customize logic
