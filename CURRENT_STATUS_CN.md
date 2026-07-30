# 当前状态与下次继续记录

这份记录用于上传仓库后，晚上重新下载工程继续排查时快速恢复上下文。

## 目标硬件

- 主板：ASUS TUF GAMING B460M-PLUS / B460M-PLUS (WI-FI) 系列
- CPU：Intel Core i7-10700F，无核显
- 显卡：蓝宝石 Radeon RX 6600 XT
- 内存：32 GB DDR4 2600 MHz
- SSD：英睿达 Crucial P5 M.2 1 TB
- 有线网卡：Intel Ethernet Connection (12) I219-V
- Wi-Fi：Intel AX200 路线已在 EFI 中准备 `itlwm.kext`
- 蓝牙：用户当前说明是单独插的 USB 蓝牙发射器；后续 USB 映射时要保留它插的那个 USB 口
- 目标系统：macOS Tahoe / macOS 26
- SMBIOS：iMac20,1

## 已写入的 SMBIOS / ROM

`EFI/OC/config.plist -> PlatformInfo -> Generic` 当前写入：

```text
SystemProductName: iMac20,1
SystemSerialNumber: C02C9DZMPN5T
MLB: C02008104GUPHC1UE
SystemUUID: AAB8BE14-29DA-4D1B-9E91-2A86FF59C44C
ROM: 3C7C3F2F89D0
```

其中 `ROM` 来自 Windows 下看到的 I219-V 有线网卡物理地址：

```text
3C-7C-3F-2F-89-D0
```

在 plist 里该 ROM 会以 base64 形式显示为：

```text
PHw/L4nQ
```

这是正常的。

## 当前 EFI 主要修改

- OpenCore 已更新到 1.0.7。
- 主配置为 `EFI/OC/config.plist`。
- `EFI/OC/config_broadcom.plist` 不是当前目标配置，不适用于 Intel AX200 / 当前 USB 蓝牙发射器路线。
- 已删除 i7-10700F 不存在的 UHD 630 核显注入：
  - `DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0)` 已移除。
- RX 6600 XT 当前 boot-args：

```text
-v debug=0x100 agdpmod=pikera
```

- 已添加/启用与目标硬件相关的 kext：
  - `NVMeFix.kext`
  - `itlwm.kext`
  - `IntelBluetoothFirmware.kext`
  - `IntelBTPatcher.kext`
  - `BlueToolFixup.kext`
- 注意：如果后续确认蓝牙发射器不是 Intel 芯片，上面 Intel 蓝牙 kext 可能不负责该外置蓝牙；是否保留需按实际蓝牙芯片再判断。

## 当前 USB 配置状态

为了排查安装器图形界面无法识别键鼠，当前已切到 USBPorts 测试模式：

```text
USBPorts.kext = Enabled
USBInjectAll.kext = Disabled
XHCI-unsupported.kext = Disabled
XhciPortLimit = false
```

这只是临时测试，不是最终方案。最终应使用 USBToolBox 在 Windows 下为本机生成真实映射：

```text
USBToolBox.kext + UTBMap.kext
```

最终 USBToolBox 方案应该是：

```text
USBToolBox.kext = Enabled
UTBMap.kext = Enabled
USBPorts.kext = Disabled
USBInjectAll.kext = Disabled
XHCI-unsupported.kext = Disabled
XhciPortLimit = false
```

## 已确认的启动现象

1. 从主板 F11 选择 `UEFI: U盘` 可以进入 OpenCore。
2. OpenCore 菜单里能看到：
   - `G (external) (dmg)`
   - `Reset NVRAM`
   - 其它 OpenCore Tools
3. 已执行过 `Reset NVRAM`。
4. 选择 `G (external) (dmg)` 后，verbose 日志能进入 macOS Recovery / 安装器后期阶段，能看到 `launchd`、`installer-progress` 等日志。
5. 后续进入图形界面，停在 `support.apple.com/mac/setup` 的鼠标/键盘提示画面。
6. 这个画面说明显卡和 Recovery 基本已经进入图形阶段，当前主要问题是 macOS 安装器没有识别 USB 键鼠输入。

## 当前结论

- 目前不像是缺少 `agdpmod=pikera`，因为当前 boot-args 已包含它。
- RX 6600 XT 基本已经能进入图形阶段；显卡不是当前第一优先级问题。
- 当前第一优先级是 USB 映射：安装器图形阶段没有识别有线键盘/鼠标。
- 用户已尝试有线键鼠、后置 USB 口、Reset NVRAM，仍卡在鼠标/键盘提示画面。
- 下一步应直接做本机 USBToolBox 映射，而不是继续猜 USB 口。

## BIOS 已设置 / 目标设置

用户已按如下方向设置或确认：

```text
VT-D: Disabled
Above 4G Decoding: Enabled
Resizable BAR: Disabled
CSM: Disabled
XHCI Hand-off: Enabled
Primary Display: PEG / PCIe（BIOS 里 Graphics Configuration 可能找不到，10700F 无核显可先忽略）
Fast Boot: Disabled
Secure Boot OS Type: Other OS
SATA Mode: AHCI
```

如 BIOS 里有：

```text
Legacy USB Support = Enabled
USB Mass Storage Driver Support = Enabled
CFG Lock = Disabled
```

建议也这样设置。

## USBToolBox 映射要点

在 Windows 下运行 USBToolBox，生成 `UTBMap.kext`。

映射原则：

- 映射的是“物理 USB 口”，不是某个设备本身。
- 只保留未来确定会使用的 USB 口即可，但至少要保留：
  - 安装 U 盘可用口
  - 键盘口
  - 鼠标口
  - 外置 USB 蓝牙发射器所在口
  - 至少一个备用 USB2 口
  - 至少一个备用 USB3 口
- 键盘/鼠标通常是 USB2 设备，对应 `HSxx`。
- USB3 U 盘/移动硬盘对应 `SSxx`。
- 一个蓝色 USB3 物理口通常要测两次：
  - 用 USB2 设备测 `HSxx`
  - 用 USB3 设备测 `SSxx`
- 外置 USB 蓝牙发射器插在哪个物理口，就保留哪个 `HSxx`，端口类型设为 `USB2`。
- 只有主板内建蓝牙、PCIe 转接卡蓝牙线接主板内置针脚、笔记本内建蓝牙这类情况，端口类型才设 `Internal`。
- 当前用户说明是单独插的 USB 蓝牙发射器，所以默认按 `USB2` 处理，不按 `Internal` 处理。

## 下次继续建议步骤

1. 在 Windows 下用 USBToolBox 扫描并生成 `UTBMap.kext`。
2. 下载/准备 `USBToolBox.kext`。
3. 放入：

```text
EFI/OC/Kexts/USBToolBox.kext
EFI/OC/Kexts/UTBMap.kext
```

4. 修改 `EFI/OC/config.plist`：

```text
启用 USBToolBox.kext
启用 UTBMap.kext
禁用 USBPorts.kext
禁用 USBInjectAll.kext
禁用 XHCI-unsupported.kext
XhciPortLimit = false
```

5. 校验 plist：

```bash
python - <<'PY'
import plistlib
plistlib.load(open("EFI/OC/config.plist", "rb"))
print("OK EFI/OC/config.plist")
PY
```

6. 复制更新后的 EFI 到 U 盘。
7. 从 U 盘启动 OpenCore，先 `Reset NVRAM`，再选 `G (external) (dmg)`。
8. 如果能通过鼠标/键盘提示页，继续安装 Tahoe。

## 安装盘说明

- gibMacOS 下载的 Tahoe 18GB `InstallAssistant.pkg` 是完整安装器包，不适合当前 gibMacOS `Select Recovery Package` 流程。
- 当前 U 盘 Recovery 路线应使用 OpenCore `macrecovery.py` 或已有的 `com.apple.recovery.boot`。
- U 盘根目录应包含：

```text
EFI/
com.apple.recovery.boot/
```
