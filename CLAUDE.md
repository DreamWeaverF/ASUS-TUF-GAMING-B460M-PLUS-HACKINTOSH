# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This repository is an OpenCore EFI configuration for an ASUS TUF GAMING B460M PLUS Hackintosh. It is not an application codebase: most maintained assets are OpenCore plist configuration, ACPI tables, EFI drivers/tools, kext bundles, and hardware documentation.

Current target hardware from the README and CURRENT_STATUS_CN.md:
- Motherboard: ASUS TUF GAMING B460M-PLUS / B460M-PLUS (WI-FI) family
- CPU: Intel Core i7-10700F, no iGPU
- GPU: Sapphire Radeon RX 6600 XT
- Memory: DDR4 2600 MHz, 32 GB
- NVMe SSD: Crucial P5 M.2 1 TB
- Ethernet: Intel I219-V; ROM currently set from MAC 3C-7C-3F-2F-89-D0
- Wi-Fi: Intel AX200 route prepared with itlwm; Bluetooth is currently described by the user as an external USB dongle
- SMBIOS: iMac20,1
- See CURRENT_STATUS_CN.md for the latest boot troubleshooting handoff; the active blocker is USB input not working in the Recovery GUI.

## Commands

There is no build system, package manager, lint script, or automated test suite in this repository.

Useful validation commands when editing plist/configuration files:

```bash
# Parse both OpenCore configs with Python's plist parser
python - <<'PY'
import plistlib
for path in ["EFI/OC/config.plist", "EFI/OC/config_broadcom.plist"]:
    plistlib.load(open(path, "rb"))
    print(f"OK {path}")
PY
```

```bash
# If OpenCore's ocvalidate tool is installed, validate each config against OpenCore's schema
ocvalidate EFI/OC/config.plist
ocvalidate EFI/OC/config_broadcom.plist
```

```bash
# If working on AppleHDA/AppleALC codec verbs, parse an ALSA codec dump
bash Audio/verbit.sh Audio/codec#0
```

For plist editing, the README recommends OCAuxiliaryTools or ProperTree. For ACPI editing/patching, use MaciASL. HackinTool is documented for diagnostics only because its built-in patches are outdated.

## High-level layout

- `EFI/BOOT/` contains the bootloader entry point (`BOOTx64.efi`).
- `EFI/OC/` is the OpenCore tree: `OpenCore.efi`, configuration plists, ACPI tables, drivers, kexts, and tools.
- `EFI/OC/config.plist` is the main OpenCore configuration.
- `EFI/OC/config_broadcom.plist` is a Broadcom/Sonoma-oriented variant. It enables IO80211FamilyLegacy/AirPortBrcmNIC-related entries and blocks `com.apple.iokit.IOSkywalkFamily`; the referenced `IOSkywalkFamily.kext` and `IO80211FamilyLegacy.kext` bundles are not present in `EFI/OC/Kexts/` in the current tree, so verify the deployment package before treating this variant as boot-ready.
- `EFI/OC/ACPI/` contains precompiled `.aml` SSDTs. The enabled set in both configs is `SSDT-EC-USBX`, `SSDT-PLUG`, `SSDT-AWAC`, `SSDT-PMCR`, and `SSDT-RHUB`; `SSDT-DMAR.aml` exists but is not currently enabled.
- `EFI/OC/Kexts/` contains bundled kernel extensions. The main config enables Lilu, VirtualSMC, WhateverGreen, AppleALC, RestrictEvents, NVMeFix, IntelMausi, SMCProcessor, SMCSuperIO, itlwm, Intel Bluetooth kexts, BlueToolFixup, and currently `USBPorts.kext` as a temporary USB test; `USBInjectAll.kext` and `XHCI-unsupported.kext` are currently disabled.
- `EFI/OC/Drivers/` contains UEFI drivers. The configs enable HfsPlus, OpenRuntime, ResetNvramEntry, and ToggleSipEntry; OpenCanopy exists but is not enabled in the current configs.
- `EFI/OC/Tools/` contains OpenCore tools. Many are enabled as auxiliary picker tools in the plist; keep plist `Misc -> Tools` entries in sync when adding/removing tool binaries.
- `Audio/` stores Realtek/AppleALC support material: codec dumps, ACPI tables, `verbit.sh`, and a short node mapping in `Audio/README.md`.
- `README.md` and `README_CN.md` are parallel English/Simplified Chinese user documentation. Update both when changing user-facing compatibility, setup, BIOS, or hardware information.
- `Archived/` contains older documentation for reference; do not treat it as the current source of truth when it conflicts with the root README files.

## OpenCore configuration notes

- Both configs use `LauncherOption` = `Full`, `HibernateMode` = `Auto`, `HideAuxiliary` = `true`, `ShowPicker` = `true`, and a 5-second picker timeout.
- Main `config.plist` uses `SecureBootModel` = `Default`, `csr-active-config` = `00000000`, and boot args `-v debug=0x100 agdpmod=pikera`.
- `config_broadcom.plist` uses `SecureBootModel` = `Disabled`, `csr-active-config` = `03080000`, and boot args `-v amfi=0x80` for the Broadcom/Sonoma path.
- The current configs include populated SMBIOS identifiers (`MLB`, ROM, serial number, UUID). If changing PlatformInfo, regenerate a consistent iMac20,1 SMBIOS set rather than mixing identifiers between configs.
- Device properties configure AppleALC layout-id `1` at `PciRoot(0x0)/Pci(0x1F,0x3)` and Intel UHD Graphics 630 at `PciRoot(0x0)/Pci(0x2,0x0)`.
- README compatibility notes: graphics acceleration, 3.5mm audio input/output, headphone auto-switching, CPU power management, NVMe power management, S3 sleep, hibernate mode 25, USB 2/3, and HiDPI are documented as working. Wi-Fi is documented as non-functional on Sonoma without OCLP/IO80211FamilyLegacy support.
- README BIOS settings expected by this EFI: VT-D disabled, Above 4G Decoding enabled, XHCI Hand-off enabled, CSM disabled, Secure Boot OS Type set to Other OS, and Wait For F1 If Error disabled.

## Change workflow

When changing EFI contents, keep the OpenCore directory and plist entries synchronized: adding/removing a kext, ACPI table, driver, or tool normally requires updating the corresponding `config*.plist` section. After plist edits, run the Python parse check above at minimum, and prefer `ocvalidate` when available.
