# Hackintosh OpenCore Configuration - AI Assistant Guide

## Project Overview
This is a Proxmox-based Hackintosh configuration using OpenCore bootloader, designed for macOS virtualization with GPU passthrough alongside Windows. The project enables running macOS on non-Apple hardware in a virtualized environment.

## Architecture & Key Components

### Core Structure
- **EFI/OC/**: Main OpenCore bootloader configuration
  - `config.plist`: Central configuration file (2490+ lines) - handles all hardware compatibility layers
  - `ACPI/`: Custom ACPI patches for hardware compatibility (8 SSDTs)
  - `Kexts/`: Kernel extensions for hardware drivers (26 kexts)
  - `Drivers/`: EFI drivers (HfsPlus, OpenCanopy, OpenRuntime, ResetNvramEntry)

### Hardware-Specific Configurations
- **AMD Radeon GPU**: Uses `SMCRadeonSensors.kext` for temperature monitoring
- **Network**: Dual ethernet support (`LucyRTL8125Ethernet.kext`, `RealtekRTL8111.kext`) + WiFi (`itlwm.kext`, `AirportBrcmFixup.kext`)
- **Audio**: `AppleALC.kext` for audio codec support
- **USB**: Multiple USB-related SSDTs and kexts for proper USB mapping
- **Bluetooth**: `IntelBluetoothFirmware.kext` + `BrcmPatchRAM` family for different chipsets

## Critical Development Patterns

### Config.plist Modification Rules
```xml
<!-- Always maintain this structure when adding ACPI patches -->
<dict>
    <key>Comment</key>
    <string>Descriptive patch name</string>
    <key>Enabled</key>
    <true/>
    <key>Path</key>
    <string>SSDT-filename.aml</string>
</dict>
```

### Kext Management via OCK_Files/
- `OCK_Files/history.json`: Tracks kext download history with product IDs
- Individual kext folders contain both original downloads and active copies
- Always sync changes between `OCK_Files/[kext]/` and `EFI/OC/Kexts/[kext]/`

### ACPI Patching Workflow
1. Custom SSDTs in `EFI/OC/ACPI/` override system DSDT
2. Binary patches in config.plist rename conflicting methods (e.g., `_CRS` → `XCRS`)
3. Original ACPI tables stored in root `ACPI/` for reference

## Essential Workflows

### Adding New Hardware Support
1. Research macOS compatibility and required kexts
2. Download kext to `OCK_Files/[kext-name]/`
3. Copy to `EFI/OC/Kexts/`
4. Add entry to config.plist `Kernel->Add` section
5. Update `OCK_Files/history.json` with product ID

### GPU Configuration (Critical for Proxmox)
- DeviceProperties section handles GPU PCI device properties
- `WhateverGreen.kext` provides graphics patches
- `ResizeAppleGpuBars` setting crucial for modern GPUs
- Test GPU passthrough functionality after any graphics changes

### Debugging Boot Issues
1. Enable OpenCore debug logging in `Misc->Debug`
2. Check ACPI patch conflicts in `ACPI->Patch` section
3. Verify kext load order and dependencies
4. Use `ResetNvramEntry.efi` to clear problematic NVRAM

## Project Conventions

### File Naming
- SSDTs: `SSDT-[Function].aml` (e.g., `SSDT-USB-Reset.aml`)
- Kexts: Original names preserved for compatibility tracking
- Comments in config.plist must match actual filenames exactly

### Version Management
- No traditional versioning - configuration evolves with hardware changes
- `OCK_Files/history.json` serves as change log for kext updates
- OpenCore version tracked by `OpenCore.efi` binary

### Safety Patterns
- Always backup `config.plist` before major changes
- Test changes in Proxmox environment before physical deployment
- Keep original ACPI dumps for reference and troubleshooting
- Maintain both working and experimental configurations

## Integration Points

### Proxmox Integration
- EFI partition must be accessible to Proxmox VM
- GPU passthrough requires specific PCI device configuration
- Network configuration must account for virtualized environment

### Cross-Platform Considerations
- Windows compatibility maintained through proper ACPI implementations
- Shared EFI system partition between macOS and Windows boots
- Hardware resource allocation coordinated through Proxmox

## Knowledge Management
[byterover-mcp]
Use `byterover-store-knowledge` when learning OpenCore configuration patterns.
Use `byterover-retrieve-knowledge` before modifying hardware compatibility settings.
