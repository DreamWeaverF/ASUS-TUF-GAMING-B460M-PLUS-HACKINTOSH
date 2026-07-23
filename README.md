# ASUS TUF GAMING B460M-PLUS (WI-FI) Hackintosh EFI

EFI configuration for ASUS TUF GAMING B460M-PLUS (WI-FI), Intel Comet Lake, and AMD RX 6600 XT.

English (current)  
[简体中文](README_CN.md)

---

## ⚠️ Disclaimer

**Your warranty is now void.**  
Please research thoroughly before using this project. I am not responsible for any loss, including but not limited to kernel panics, device boot failures, hardware malfunction, storage damage, data loss, and other unforeseen consequences.

---

## 🖥️ Target Hardware

| Specification | Detail |
|:--------------|:-------|
| Motherboard | ASUS TUF GAMING B460M-PLUS (WI-FI) |
| BIOS Version | Not fixed; configure BIOS as listed below |
| CPU | Intel Core i7-10700F |
| Integrated Graphics | None; F-series CPU |
| Discrete Graphics | Sapphire Radeon RX 6600 XT |
| Memory | DDR4 2600 MHz, 32 GB |
| NVMe SSD | Crucial P5 M.2 1 TB |
| Ethernet | Onboard Intel I219-V, via IntelMausi.kext |
| Wi-Fi / Bluetooth | Onboard Intel AX200 |
| Audio | Onboard Realtek S1200A / ALC1220-family, AppleALC layout-id 1 |
| SMBIOS | iMac20,1 |
| OpenCore | 1.0.7 |
| Target macOS | macOS Tahoe |

This EFI is based on the original B460M-PLUS configuration but is now adapted for a dGPU-only i7-10700F + RX 6600 XT setup. The Intel UHD 630 device property injection has been removed.

---

## ✅ Current Support Matrix

Status meanings:

- ✅ Configured in the current EFI
- 🟡 Configured, but requires real-machine verification or post-install work
- ❌ Not included / not recommended in the current EFI

### Boot, Graphics & Display

| Feature | Status | Dependency / Setting | Remarks |
|--------|:------:|:---------------------|---------|
| macOS Tahoe installer boot | 🟡 | OpenCore 1.0.7, current Acidanthera kexts | Prepared for Tahoe bring-up; verify on the target machine |
| RX 6600 XT graphics acceleration | ✅ | WhateverGreen.kext, `agdpmod=pikera` | Use the RX 6600 XT as the only display output |
| Intel UHD Graphics | ❌ | — | i7-10700F has no iGPU; no framebuffer injection is configured |
| HiDPI / external display | 🟡 | RX 6600 XT | Depends on monitor and port; DisplayPort is recommended for first boot |

### Network & Bluetooth

| Feature | Status | Dependency / Setting | Remarks |
|--------|:------:|:---------------------|---------|
| Ethernet | ✅ | IntelMausi.kext | Use wired Ethernet for installation and first boot |
| Intel AX200 Wi-Fi | 🟡 | itlwm.kext + HeliPort | Not native AirportItlwm; install HeliPort after macOS boots |
| Intel AX200 Bluetooth | 🟡 | IntelBluetoothFirmware.kext, IntelBTPatcher.kext, BlueToolFixup.kext | Final stability depends on correct USB mapping for the internal Bluetooth port |
| Broadcom / OCLP Wi-Fi path | ❌ | — | `config_broadcom.plist` is not the target config for this hardware |

### Audio, Storage, USB & Power

| Feature | Status | Dependency / Setting | Remarks |
|--------|:------:|:---------------------|---------|
| Onboard audio | 🟡 | AppleALC.kext, layout-id 1 | Test rear/front output and microphone after boot |
| Crucial P5 NVMe | ✅ | NVMeFix.kext | The SSD is not one of the README's previously listed incompatible models |
| USB installer bring-up | 🟡 | USBInjectAll.kext, XHCI-unsupported.kext, `XhciPortLimit=true` | Temporary first-boot setup only |
| Final USB mapping | ❌ | USBToolBox + UTBMap.kext | Generate a map on this exact board after boot; include the AX200 Bluetooth internal USB port |
| Sleep / wake | 🟡 | ACPI + final USB map | Do not judge sleep until USB mapping is finalized |
| Hibernate mode 25 | ❌ | — | Not recommended during initial bring-up |

---

## 📁 Configuration Notes

- Use [EFI/OC/config.plist](EFI/OC/config.plist) as the active configuration.
- [EFI/OC/config_broadcom.plist](EFI/OC/config_broadcom.plist) is kept only as a non-target/legacy variant and should not be used for the Intel AX200 setup.
- Current boot args:

```text
-v debug=0x100 agdpmod=pikera
```

- Current enabled ACPI tables:
  - `SSDT-EC-USBX.aml`
  - `SSDT-PLUG.aml`
  - `SSDT-AWAC.aml`
  - `SSDT-PMCR.aml`
  - `SSDT-RHUB.aml`
- `SSDT-DMAR.aml` exists but is not enabled.
- `USBPorts.kext` exists but is disabled because it is not a final map for the B460M-PLUS (WI-FI) board.
- The EFI currently uses temporary SMBIOS values. Before signing into Apple ID/iCloud, regenerate iMac20,1 SMBIOS values and set `ROM` from your own Ethernet MAC address.

---

## 🛠️ Requirements

- USB flash drive for the macOS installer
- [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools) or [ProperTree](https://github.com/corpnewt/ProperTree) for plist editing
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) or another trusted SMBIOS generator
- [USBToolBox](https://github.com/USBToolBox/tool) for final USB mapping
- [HeliPort](https://github.com/OpenIntelWireless/HeliPort) for Wi-Fi when using `itlwm.kext`
- [MaciASL](https://github.com/acidanthera/MaciASL) only if ACPI edits are needed
- [HackinTool](https://github.com/headkaze/Hackintool) for diagnostics only; do not rely on its old built-in patches

---

## BIOS Settings

Set these before first boot:

- Advanced > System Agent (SA) Configuration > VT-D: **Disabled**
- Advanced > System Agent (SA) Configuration > Above 4G Decoding: **Enabled**
- Advanced > USB Configuration > XHCI Hand-off: **Enabled**
- Boot > CSM (Compatibility Support Module) > Launch CSM: **Disabled**
- Boot > Secure Boot > OS Type: **Other OS**
- Boot > Boot Configuration > Wait For 'F1' If Error: **Disabled**
- Primary Display / Initial Display Output: **PEG / PCIe**
- Resizable BAR: **Disabled**, if present
- SATA Mode: **AHCI**
- Fast Boot: **Disabled**
- CFG Lock: **Disabled**, if present

Connect the monitor to the RX 6600 XT. DisplayPort is recommended for first boot troubleshooting.

---

## Installation Notes

1. Create the macOS Tahoe installer USB.
2. Mount the USB EFI partition.
3. Copy the repository's [EFI/](EFI/) folder to the root of the USB EFI partition.
4. Boot from the USB and run **Reset NVRAM** in the OpenCore picker after changing SMBIOS/NVRAM values.
5. Boot the installer in verbose mode.
6. Use wired Ethernet for installation.
7. After macOS boots, install HeliPort if Wi-Fi is managed through `itlwm.kext`.
8. Generate a final USB map with USBToolBox and replace the temporary USB setup.

Expected EFI partition layout:

```text
EFI partition/
└── EFI/
    ├── BOOT/
    │   └── BOOTx64.efi
    └── OC/
        ├── OpenCore.efi
        ├── config.plist
        ├── ACPI/
        ├── Drivers/
        ├── Kexts/
        └── Tools/
```

---

## Validation

After editing OpenCore configuration, validate the plist:

```bash
python - <<'PY'
import plistlib
for path in ["EFI/OC/config.plist"]:
    plistlib.load(open(path, "rb"))
    print(f"OK {path}")
PY
```

If `ocvalidate` from the matching OpenCore release is available:

```bash
ocvalidate EFI/OC/config.plist
```

---

## References

- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [Dortania OpenCore Post Install Guide](https://dortania.github.io/OpenCore-Post-Install/)
- [OpenIntelWireless itlwm / AirportItlwm](https://github.com/OpenIntelWireless/itlwm)
- [OpenIntelWireless IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [USBToolBox](https://github.com/USBToolBox/tool)

---

## Notice

This repository is for sharing and assisting with Hackintosh installations. Commercial use is not permitted.

Released under the MIT License.
