# ASUS TUF GAMING B460M-PLUS (WI-FI) Hackintosh EFI

适用于 ASUS TUF GAMING B460M-PLUS (WI-FI)、Intel Comet Lake 平台和 AMD RX 6600 XT 的 OpenCore EFI 配置。

简体中文（当前）  
[English](README.md)

---

## ⚠️ 免责声明

**您的保修已失效。**  
在使用本项目前请务必充分了解相关内容。本人不对任何损失负责，包括但不限于：内核崩溃、设备无法启动、硬件故障、存储损坏、数据丢失及其他不可预见的后果。

---

## 🖥️ 目标硬件

| 规格 | 详情 |
|:-----|:-----|
| 主板 | ASUS TUF GAMING B460M-PLUS (WI-FI) |
| BIOS 版本 | 不固定；请按下方 BIOS 设置调整 |
| CPU | Intel Core i7-10700F |
| 核显 | 无；F 系列 CPU |
| 独显 | 蓝宝石 Radeon RX 6600 XT |
| 内存 | DDR4 2600 MHz，32 GB |
| NVMe SSD | 英睿达 Crucial P5 M.2 1 TB |
| 有线网卡 | 板载 Intel I219-V，通过 IntelMausi.kext 驱动 |
| 无线 / 蓝牙 | 板载 Intel AX200 |
| 声卡 | 板载 Realtek S1200A / ALC1220 系列，AppleALC layout-id 1 |
| SMBIOS | iMac20,1 |
| OpenCore | 1.0.7 |
| 目标 macOS | macOS Tahoe |

本 EFI 基于原 B460M-PLUS 配置修改，现在已适配 **i7-10700F 无核显 + RX 6600 XT 独显**。原来的 Intel UHD 630 设备属性注入已经移除。

---

## ✅ 当前支持状态

状态说明：

- ✅ 当前 EFI 已配置
- 🟡 已配置，但需要真机验证或安装后处理
- ❌ 当前 EFI 不包含 / 不推荐

### 启动、显卡与显示

| 功能 | 状态 | 依赖 / 设置 | 备注 |
|------|:----:|:------------|------|
| macOS Tahoe 安装器启动 | 🟡 | OpenCore 1.0.7、当前 Acidanthera kext | 已按 Tahoe 首启方向配置，需在目标机器实测 |
| RX 6600 XT 图形加速 | ✅ | WhateverGreen.kext、`agdpmod=pikera` | 只使用 RX 6600 XT 输出显示 |
| Intel UHD Graphics | ❌ | — | i7-10700F 没有核显，未配置 framebuffer 注入 |
| HiDPI / 外接显示 | 🟡 | RX 6600 XT | 取决于显示器和接口；首启排错建议优先用 DisplayPort |

### 网络与蓝牙

| 功能 | 状态 | 依赖 / 设置 | 备注 |
|------|:----:|:------------|------|
| 有线网卡 | ✅ | IntelMausi.kext | 安装和首启建议优先使用有线网络 |
| Intel AX200 Wi-Fi | 🟡 | itlwm.kext + HeliPort | 不是原生 AirportItlwm 方案；进系统后安装 HeliPort 使用 |
| Intel AX200 蓝牙 | 🟡 | IntelBluetoothFirmware.kext、IntelBTPatcher.kext、BlueToolFixup.kext | 稳定性取决于后续 USB 定制是否包含蓝牙内建 USB 端口 |
| Broadcom / OCLP 无线路线 | ❌ | — | `config_broadcom.plist` 不是本机 Intel AX200 的目标配置 |

### 声卡、硬盘、USB 与电源

| 功能 | 状态 | 依赖 / 设置 | 备注 |
|------|:----:|:------------|------|
| 板载声卡 | 🟡 | AppleALC.kext、layout-id 1 | 启动后测试前后音频输出和麦克风 |
| 英睿达 P5 NVMe | ✅ | NVMeFix.kext | 不属于原 README 提到的不兼容 SSD 型号 |
| USB 安装器首启 | 🟡 | USBInjectAll.kext、XHCI-unsupported.kext、`XhciPortLimit=true` | 仅作为临时首启方案 |
| 最终 USB 定制 | ❌ | USBToolBox + UTBMap.kext | 需要在这块主板上重新扫端口，必须包含 AX200 蓝牙内建 USB 端口 |
| 睡眠 / 唤醒 | 🟡 | ACPI + 最终 USB 定制 | USB 定制完成前不建议判断睡眠稳定性 |
| 休眠模式 25 | ❌ | — | 首启和调试阶段不推荐启用 |

---

## 📁 配置说明

- 当前目标配置文件是 [EFI/OC/config.plist](EFI/OC/config.plist)。
- [EFI/OC/config_broadcom.plist](EFI/OC/config_broadcom.plist) 仅保留为非目标/旧配置，不适用于当前 Intel AX200 方案。
- 当前启动参数：

```text
-v debug=0x100 agdpmod=pikera
```

- 当前启用的 ACPI：
  - `SSDT-EC-USBX.aml`
  - `SSDT-PLUG.aml`
  - `SSDT-AWAC.aml`
  - `SSDT-PMCR.aml`
  - `SSDT-RHUB.aml`
- `SSDT-DMAR.aml` 存在，但当前未启用。
- `USBPorts.kext` 存在，但当前禁用，因为它不是 B460M-PLUS (WI-FI) 的最终 USB 定制。
- 当前 EFI 使用的是临时生成的 SMBIOS。登录 Apple ID / iCloud 前，请重新生成 iMac20,1 三码，并把 `ROM` 设置为你自己的有线网卡 MAC 地址。

---

## 🛠️ 准备工具

- 用于制作 macOS 安装器的 U 盘
- [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools) 或 [ProperTree](https://github.com/corpnewt/ProperTree)，用于编辑 plist
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 或其他可信工具，用于生成三码
- [USBToolBox](https://github.com/USBToolBox/tool)，用于最终 USB 定制
- [HeliPort](https://github.com/OpenIntelWireless/HeliPort)，用于配合 `itlwm.kext` 管理 Wi-Fi
- [MaciASL](https://github.com/acidanthera/MaciASL)，仅在需要修改 ACPI 时使用
- [HackinTool](https://github.com/headkaze/Hackintool) 仅用于诊断，不建议使用其过时的内置补丁

---

## BIOS 设置

首次启动前请设置：

- Advanced > System Agent (SA) Configuration > VT-D：**Disabled**
- Advanced > System Agent (SA) Configuration > Above 4G Decoding：**Enabled**
- Advanced > USB Configuration > XHCI Hand-off：**Enabled**
- Boot > CSM (Compatibility Support Module) > Launch CSM：**Disabled**
- Boot > Secure Boot > OS Type：**Other OS**
- Boot > Boot Configuration > Wait For 'F1' If Error：**Disabled**
- Primary Display / Initial Display Output：**PEG / PCIe**
- Resizable BAR：**Disabled**，如果 BIOS 里有该选项
- SATA Mode：**AHCI**
- Fast Boot：**Disabled**
- CFG Lock：**Disabled**，如果 BIOS 里有该选项

显示器请连接到 RX 6600 XT。首启排查黑屏时，建议优先使用 DisplayPort。

---

## 安装说明

1. 制作 macOS Tahoe 安装 U 盘。
2. 挂载 U 盘 EFI 分区。
3. 把本仓库的 [EFI/](EFI/) 文件夹复制到 U 盘 EFI 分区根目录。
4. 从 U 盘启动；修改 SMBIOS/NVRAM 后，在 OpenCore 菜单执行一次 **Reset NVRAM**。
5. 使用 verbose 模式启动安装器。
6. 安装阶段优先使用有线网络。
7. 如果 Wi-Fi 使用 `itlwm.kext`，进入系统后安装 HeliPort。
8. 安装后用 USBToolBox 生成最终 USB 定制，替换当前临时 USB 方案。

正确的 EFI 分区结构：

```text
EFI 分区/
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

## 验证

修改 OpenCore 配置后，先检查 plist 语法：

```bash
python - <<'PY'
import plistlib
for path in ["EFI/OC/config.plist"]:
    plistlib.load(open(path, "rb"))
    print(f"OK {path}")
PY
```

如果有当前 OpenCore 版本配套的 `ocvalidate`：

```bash
ocvalidate EFI/OC/config.plist
```

---

## 参考资料

- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [Dortania OpenCore Post Install Guide](https://dortania.github.io/OpenCore-Post-Install/)
- [OpenIntelWireless itlwm / AirportItlwm](https://github.com/OpenIntelWireless/itlwm)
- [OpenIntelWireless IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [USBToolBox](https://github.com/USBToolBox/tool)

---

## 注意

本仓库仅用于分享和协助黑苹果安装。禁止商业用途。

以 MIT 许可证发布。
