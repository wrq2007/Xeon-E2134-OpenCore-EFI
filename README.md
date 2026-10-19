# Xeon E-2134 Hackintosh EFI

**语言 / Language**：[中文说明](#中文说明) · [English](#english)

---

<a id="中文说明"></a>

# Xeon E-2134 黑苹果 EFI（OpenCore 1.0.9 / macOS 15.8 Sequoia）

> 昂达 9D3-VH + Xeon E-2134 + RX 570 的 OpenCore EFI 配置，SMBIOS 使用 MacPro7,1，与 Windows 双系统共存。
>
> **注意：本仓库的三码（序列号 / MLB / SystemUUID）与 ROM 已移除，必须填入自己的才能正常激活 iMessage / FaceTime。**

## 硬件配置

| 组件 | 型号 | 备注 |
|---|---|---|
| 主板 | 昂达（ONDA）9D3-VH | LPC 控制器 PCI ID `8086:1901`（Intel C230 系列芯片组） |
| CPU | Intel Xeon E-2134 | 4 核 8 线程，基础频率 3.5 GHz，睿频 4.5 GHz，TDP 71W |
| 显卡 | AMD Radeon RX 570 8 GB | PCI ID `1002:67df`（Polaris 20 XL），macOS 原生支持 |
| 内存 | 16 GB | |
| 系统盘 | SanDisk SSD PLUS 480 GB | macOS 与 Windows 双系统共存于同一硬盘 |
| 有线网卡 | Realtek RTL8111 | PCI ID `10ec:8168` |
| 无线网卡 | Intel Wi-Fi 6 AX201 | PCI ID `8086:2723`，itlwm 2.3.0（需配合 HeliPort 连接 Wi-Fi） |
| 蓝牙 | Intel AX201 蓝牙 | IntelBluetoothFirmware + IntelBTPatcher + BlueToolFixup |
| 声卡 | 板载 Intel HD Audio | AppleALC，`layout-id = 1` |
| SMBIOS | MacPro7,1 | `UpdateSMBIOSMode = Custom` |

## 软件环境

| 项目 | 版本 |
|---|---|
| macOS | 15.8（Build 24H23）Sequoia |
| OpenCore | 1.0.9 |
| BIOS | 昂达 9D3-VH 主板 BIOS |
| 引导入口 | `EFI/BOOT/BOOTX64.efi`（OpenCanopy 图形界面） |

## EFI 目录结构

```
EFI/
├── BOOT/
│   ├── BOOTX64.efi               # OpenCore 引导器
│   └── bootx64.efi.win-backup    # Windows 引导备份（双系统用，勿删）
└── OC/
    ├── ACPI/                     # 6 个 SSDT
    ├── Drivers/                  # 4 个 UEFI 驱动
    ├── Kexts/                    # 15 个 kext
    ├── Resources/                # OpenCanopy 主题资源
    ├── Tools/
    ├── OpenCore.efi
    └── config.plist
```

> 本仓库**不包含** `EFI/Microsoft/`（Windows 引导与恢复目录）。双系统用户请勿删除自己 EFI 分区里的该目录，否则 Windows 会失联。

## Kext 清单（15 个）

| Kext | 版本 | 作用 |
|---|---|---|
| Lilu | 1.7.3 | 补丁框架，多数 kext 的公共依赖 |
| VirtualSMC | 1.3.9 | 模拟 SMC，黑苹果核心组件 |
| SMCProcessor | 1.3.9 | CPU 温度/频率传感器 |
| SMCSuperIO | 1.3.9 | 风扇、电压传感器 |
| WhateverGreen | 1.7.2 | 显卡补丁与硬件加速 |
| AppleALC | 1.9.9 | 声卡驱动（`layout-id = 1`） |
| RealtekRTL8111 | 2.4.2 | Realtek 有线网卡驱动 |
| itlwm | 2.3.0 | Intel AX201 无线网卡驱动 |
| IntelBluetoothFirmware | 2.5.0 | Intel 蓝牙固件 |
| IntelBTPatcher | 2.5.0 | macOS 12+ Intel 蓝牙补丁 |
| BlueToolFixup | 2.7.3 | macOS Monterey+ 蓝牙兼容修复 |
| USBToolBox | 1.2.0 | USB 端口定制工具 |
| UTBMap | 1.1 | 本机 USB 端口映射（与 USBToolBox 配套） |
| RestrictEvents | 1.1.7 | 处理器/内存信息修正，避免系统信息异常 |
| XHCI-unsupported | 0.9.2 | 为 macOS 不原生支持的 USB 控制器启用 USB 3.0 |

## ACPI 与 UEFI 驱动

### SSDT（`EFI/OC/ACPI/`）

| 文件 | 作用 |
|---|---|
| `SSDT-EC.aml` | 嵌入式控制器（伪 EC） |
| `SSDT-PLUG.aml` | 注入 CPU 电源管理（plugin-type，XCPM 变频必需） |
| `SSDT-PMC.aml` | 300 系列主板 NVRAM 支持 |
| `SSDT-RTCAWAC.aml` | RTC / AWAC 兼容 |
| `SSDT-SBUS.aml` | SMBus 支持 |
| `SSDT-USBX.aml` | USB 供电电流限制 |

`config.plist` 中 `ACPI → Patch` 为空，无 ACPI 重命名补丁。

### UEFI 驱动（`EFI/OC/Drivers/`）

| 文件 | 作用 |
|---|---|
| `OpenRuntime.efi` | 必需，NVRAM 读写与内存布局修正 |
| `OpenCanopy.efi` | 图形化引导主题 |
| `HfsPlus.efi` | HFS+ 文件系统支持 |
| `ResetNvramEntry.efi` | 在引导菜单中提供"重置 NVRAM"入口 |

## config.plist 要点

- **启动参数**：`boot-args` 为空，未附加任何参数。
- **设备属性**（`DeviceProperties → Add`）：
  - `PciRoot(0x0)/Pci(0x1f,0x3)` → `layout-id = 1`（声卡）
  - `PciRoot(0x0)/Pci(0x1D,0x0)/Pci(0x0,0x0)` → `built-in`（将 AX201 无线网卡标记为内置设备）
- **安全设置**（`Misc → Security`）：
  - `SecureBootModel = Disabled`
  - `Vault = Optional`
  - `ScanPolicy = 0`（显示所有可引导系统，含 Windows）
  - `DmgLoading = Signed`
  - `EnablePassword = false`（未设置开机密码）
- **平台信息**：`PlatformInfo → UpdateSMBIOSMode = Custom`，`SystemProductName = MacPro7,1`

## 使用前必读（重要）

1. **填入自己的三码**。用 ProperTree 或 OCSimpleTool 打开 `EFI/OC/config.plist`，在 `PlatformInfo → Generic` 中补上：
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`
   - `ROM`（可选；不填则自动取有线网卡 MAC）

   三个值必须成套生成，不能混搭。`SystemProductName` 保持 `MacPro7,1` 不要动。

2. **验证序列号是否撞车**。去 [checkcoverage.apple.com](https://checkcoverage.apple.com) 查询你填的序列号：
   - 显示"序列号无效 / 请验证购买日期" → **理想状态**，没撞真机；
   - 显示具体购买日期和保修到期日 → **撞了别人的真机**，iMessage / FaceTime 会激活失败，必须换一个。

3. **先备份再替换**。首次使用时先备份原机能启动的 EFI，再整目录覆盖。

4. 编辑 config.plist 时保持 XML 格式，推荐工具：
   - [ProperTree](https://github.com/corpnewt/ProperTree)（跨平台）
   - OCSimpleTool（macOS）

## 已验证状态

| 项目 | 状态 | 说明 |
|---|---|---|
| CPU 电源管理 / 变频 | ✅ 正常 | XCPM 生效。`powermetrics` 实测：全核满载 **4.2 GHz**（120% nominal），单核 4.02 GHz，Package 功耗 48.9W（TDP 71W），限制原因仅 `MAX_TURBO_LIMIT`（已达睿频上限，无降频） |
| 启动 / 引导 | ✅ 正常 | OpenCanopy 界面，双系统切换正常 |
| 显卡 | ⚠️ 待补充 | RX 570 为 macOS 原生支持型号 |
| 声卡 / 有线网 / 无线 / 蓝牙 | ⚠️ 待补充 | |
| 睡眠 / iMessage / FaceTime / 接力 | ⚠️ 待补充 | |

> ✅ = 本机实测通过；⚠️ = 请自行验证后补充。本机**未使用 CPUFriend**，原生 XCPM 调度已正常。

## BIOS 设置

> 以昂达 9D3-VH 实际 BIOS 版本为准，通用建议如下。

**关闭**

- Fast Boot（快速启动）
- CSM（兼容性支持模块）
- Secure Boot（安全启动）
- CFG Lock —— 若 BIOS 无法关闭，需在 `config.plist` 的 `Kernel → Quirks` 中启用 `AppleXcpmCfgLock` 与 `AppleCpuPmCfgLock`
- Intel SGX
- VT-d（或配合 `Kernel → Quirks → DisableIoMapper`）

**开启**

- Above 4G Decoding
- XHCI / EHCI Hand-off
- 启动模式设为 UEFI Only

## 常见问题

- **开机卡在 `Waiting for Root Device`**：多为 USB 定制失效，检查 `UTBMap.kext` 是否与当前主板匹配，必要时重新生成。
- **无限重启 / 五国**：先进引导菜单选 Reset NVRAM 重置，再检查三码是否填错或格式非法。
- **关于本机序列号显示为空**：三码还没填，见[使用前必读](#使用前必读重要)。
- **Wi-Fi 无法连接**：itlwm 2.3.0 需配合 [HeliPort](https://github.com/OpenIntelWireless/HeliPort) 使用；若想原生 Wi-Fi 菜单，可换用 itlwm 的 `v3.0.0` alpha 分支或 OpenIntelWireless 方案。
- **CPU 频率上不去**：先看 `Misc → Security` 与三码是否完整；本机原生 XCPM 已能跑满睿频，不需要 CPUFriend。

## 脱敏说明

推送前已从 `config.plist` 中**删除**以下敏感项：

| 键 | 位置 | 说明 |
|---|---|---|
| `SystemSerialNumber` | `PlatformInfo → Generic` | 序列号 |
| `MLB` | `PlatformInfo → Generic` | 主板序列号 |
| `SystemUUID` | `PlatformInfo → Generic` | 硬件 UUID |
| `ROM` | `PlatformInfo → Generic` | 网卡 MAC 地址 |

同时排除了以下文件/目录：

- `EFI/Microsoft/` —— Windows 引导与恢复目录（属 Windows 系统文件，与黑苹果配置无关）
- `config.plist.bak-*` —— 原机上的配置备份文件

除上述外，`config.plist` 其余内容与机器上使用的完全一致，可直接填入三码后使用。

## 致谢与声明

- [OpenCore Install Guide（Dortania）](https://dortania.github.io/OpenCore-Install-Guide/)
- [Acidanthera](https://github.com/acidanthera)：Lilu / VirtualSMC / WhateverGreen / AppleALC 等
- [OpenIntelWireless](https://github.com/OpenIntelWireless)：itlwm / IntelBluetoothFirmware
- [USBToolBox](https://github.com/USBToolBox) 与 [corpnewt](https://github.com/corpnewt) 工具链

本 EFI 配置仅供学习交流使用，风险自负。请勿使用他人三码进行激活或商业用途。

---

<a id="english"></a>

# Xeon E-2134 Hackintosh EFI (OpenCore 1.0.9 / macOS 15.8 Sequoia)

> OpenCore EFI for an ONDA 9D3-VH + Xeon E-2134 + RX 570 build. SMBIOS: MacPro7,1. Dual-boots with Windows.
>
> **Note: The serial number, MLB, SystemUUID and ROM have been removed. You must fill in your own before iMessage / FaceTime can be activated.**

## Hardware

| Component | Model | Notes |
|---|---|---|
| Motherboard | ONDA 9D3-VH | LPC controller PCI ID `8086:1901` (Intel C230 series chipset) |
| CPU | Intel Xeon E-2134 | 4C/8T, 3.5 GHz base, 4.5 GHz turbo, TDP 71W |
| GPU | AMD Radeon RX 570 8 GB | PCI ID `1002:67df` (Polaris 20 XL), natively supported |
| RAM | 16 GB | |
| System disk | SanDisk SSD PLUS 480 GB | macOS and Windows dual-boot on the same drive |
| Ethernet | Realtek RTL8111 | PCI ID `10ec:8168` |
| Wi-Fi | Intel Wi-Fi 6 AX201 | PCI ID `8086:2723`, itlwm 2.3.0 (needs HeliPort to connect) |
| Bluetooth | Intel AX201 Bluetooth | IntelBluetoothFirmware + IntelBTPatcher + BlueToolFixup |
| Audio | Onboard Intel HD Audio | AppleALC, `layout-id = 1` |
| SMBIOS | MacPro7,1 | `UpdateSMBIOSMode = Custom` |

## Software

| Item | Version |
|---|---|
| macOS | 15.8 (Build 24H23) Sequoia |
| OpenCore | 1.0.9 |
| BIOS | ONDA 9D3-VH motherboard BIOS |
| Boot entry | `EFI/BOOT/BOOTX64.efi` (OpenCanopy GUI) |

## EFI Directory Structure

```
EFI/
├── BOOT/
│   ├── BOOTX64.efi               # OpenCore bootloader
│   └── bootx64.efi.win-backup    # Windows bootloader backup (dual-boot, do not delete)
└── OC/
    ├── ACPI/                     # 6 SSDTs
    ├── Drivers/                  # 4 UEFI drivers
    ├── Kexts/                    # 15 kexts
    ├── Resources/                # OpenCanopy theme resources
    ├── Tools/
    ├── OpenCore.efi
    └── config.plist
```

> This repo does **not** include `EFI/Microsoft/` (Windows boot and recovery directory). Dual-boot users must keep that directory on their own EFI partition, or Windows will become unbootable.

## Kext List (15)

| Kext | Version | Purpose |
|---|---|---|
| Lilu | 1.7.3 | Patch framework, common dependency of most kexts |
| VirtualSMC | 1.3.9 | Emulated SMC, core hackintosh component |
| SMCProcessor | 1.3.9 | CPU temperature/frequency sensors |
| SMCSuperIO | 1.3.9 | Fan and voltage sensors |
| WhateverGreen | 1.7.2 | GPU patching and hardware acceleration |
| AppleALC | 1.9.9 | Audio driver (`layout-id = 1`) |
| RealtekRTL8111 | 2.4.2 | Realtek Ethernet driver |
| itlwm | 2.3.0 | Intel AX201 Wi-Fi driver |
| IntelBluetoothFirmware | 2.5.0 | Intel Bluetooth firmware |
| IntelBTPatcher | 2.5.0 | Intel Bluetooth patch for macOS 12+ |
| BlueToolFixup | 2.7.3 | Bluetooth compatibility fix for macOS Monterey+ |
| USBToolBox | 1.2.0 | USB port customization tool |
| UTBMap | 1.1 | USB port map for this machine (paired with USBToolBox) |
| RestrictEvents | 1.1.7 | CPU/memory info correction |
| XHCI-unsupported | 0.9.2 | Enables USB 3.0 for controllers not natively supported by macOS |

## ACPI and UEFI Drivers

### SSDTs (`EFI/OC/ACPI/`)

| File | Purpose |
|---|---|
| `SSDT-EC.aml` | Embedded controller (fake EC) |
| `SSDT-PLUG.aml` | Injects CPU power management (plugin-type, required for XCPM) |
| `SSDT-PMC.aml` | NVRAM support for 300-series chipsets |
| `SSDT-RTCAWAC.aml` | RTC / AWAC compatibility |
| `SSDT-SBUS.aml` | SMBus support |
| `SSDT-USBX.aml` | USB power current limits |

`ACPI → Patch` in config.plist is empty — no ACPI renames.

### UEFI Drivers (`EFI/OC/Drivers/`)

| File | Purpose |
|---|---|
| `OpenRuntime.efi` | Required. NVRAM access and memory layout fixes |
| `OpenCanopy.efi` | Graphical boot theme |
| `HfsPlus.efi` | HFS+ filesystem support |
| `ResetNvramEntry.efi` | Adds a "Reset NVRAM" entry to the boot picker |

## config.plist Highlights

- **Boot args**: `boot-args` is empty, no extra arguments.
- **Device properties** (`DeviceProperties → Add`):
  - `PciRoot(0x0)/Pci(0x1f,0x3)` → `layout-id = 1` (audio)
  - `PciRoot(0x0)/Pci(0x1D,0x0)/Pci(0x0,0x0)` → `built-in` (marks the AX201 Wi-Fi card as built-in)
- **Security settings** (`Misc → Security`):
  - `SecureBootModel = Disabled`
  - `Vault = Optional`
  - `ScanPolicy = 0` (shows all bootable systems, including Windows)
  - `DmgLoading = Signed`
  - `EnablePassword = false` (no boot password set)
- **Platform info**: `PlatformInfo → UpdateSMBIOSMode = Custom`, `SystemProductName = MacPro7,1`

## Before You Start (Important)

1. **Fill in your own SMBIOS values.** Open `EFI/OC/config.plist` with ProperTree or OCSimpleTool and add the following under `PlatformInfo → Generic`:
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`
   - `ROM` (optional; if omitted, the Ethernet MAC is used)

   The three values must be generated as a set — do not mix and match. Leave `SystemProductName` as `MacPro7,1`.

2. **Check your serial number for collisions.** Look up your serial at [checkcoverage.apple.com](https://checkcoverage.apple.com):
   - "Serial number not valid / please verify your purchase date" → **ideal**, it does not collide with any real machine;
   - A specific purchase date and warranty expiry → **collides with a real machine**; iMessage / FaceTime will fail to activate. Generate a new one.

3. **Back up before replacing.** Back up the currently working EFI before overwriting with this one.

4. Keep config.plist in XML format. Recommended tools:
   - [ProperTree](https://github.com/corpnewt/ProperTree) (cross-platform)
   - OCSimpleTool (macOS)

## Verified Status

| Item | Status | Notes |
|---|---|---|
| CPU power management / frequency scaling | ✅ Working | XCPM active. Measured with `powermetrics`: all-core turbo **4.2 GHz** under full load (120% nominal), 4.02 GHz single-core, package power 48.9W (TDP 71W), only `MAX_TURBO_LIMIT` (at turbo ceiling, no throttling) |
| Boot / bootloader | ✅ Working | OpenCanopy GUI, dual-boot switching works |
| GPU | ⚠️ To be filled in | RX 570 is natively supported |
| Audio / Ethernet / Wi-Fi / Bluetooth | ⚠️ To be filled in | |
| Sleep / iMessage / FaceTime / Handoff | ⚠️ To be filled in | |

> ✅ = verified on this machine; ⚠️ = please verify and fill in. **CPUFriend is not used** — native XCPM scheduling already works correctly.

## BIOS Settings

> Refer to the actual BIOS of your ONDA 9D3-VH. Generic recommendations:

**Disable**

- Fast Boot
- CSM (Compatibility Support Module)
- Secure Boot
- CFG Lock — if it cannot be disabled in BIOS, enable `AppleXcpmCfgLock` and `AppleCpuPmCfgLock` under `Kernel → Quirks` in config.plist
- Intel SGX
- VT-d (or pair with `Kernel → Quirks → DisableIoMapper`)

**Enable**

- Above 4G Decoding
- XHCI / EHCI Hand-off
- UEFI Only boot mode

## Troubleshooting

- **Stuck at `Waiting for Root Device`**: usually a broken USB map. Check whether `UTBMap.kext` matches this motherboard; regenerate if needed.
- **Boot loop / kernel panic**: select Reset NVRAM in the boot picker first, then verify your SMBIOS values are correct and well-formed.
- **Serial number shows empty in About This Mac**: SMBIOS values are not filled in yet — see [Before You Start](#before-you-start-important).
- **Wi-Fi cannot connect**: itlwm 2.3.0 requires [HeliPort](https://github.com/OpenIntelWireless/HeliPort); for the native Wi-Fi menu, switch to the itlwm `v3.0.0` alpha branch or the OpenIntelWireless stack.
- **CPU frequency stays low**: check `Misc → Security` and that the SMBIOS values are complete. Native XCPM already reaches full turbo on this machine — CPUFriend is not needed.

## Sanitization Notes

The following sensitive entries were **removed** from `config.plist` before publishing:

| Key | Location | Description |
|---|---|---|
| `SystemSerialNumber` | `PlatformInfo → Generic` | Serial number |
| `MLB` | `PlatformInfo → Generic` | Main logic board serial |
| `SystemUUID` | `PlatformInfo → Generic` | Hardware UUID |
| `ROM` | `PlatformInfo → Generic` | Network MAC address |

The following files/directories were also excluded:

- `EFI/Microsoft/` — Windows boot and recovery directory (Windows system files, unrelated to the hackintosh config)
- `config.plist.bak-*` — config backups from the original machine

Apart from the above, `config.plist` is identical to the one in daily use and works once your own SMBIOS values are filled in.

## Credits and Disclaimer

- [OpenCore Install Guide (Dortania)](https://dortania.github.io/OpenCore-Install-Guide/)
- [Acidanthera](https://github.com/acidanthera): Lilu / VirtualSMC / WhateverGreen / AppleALC, etc.
- [OpenIntelWireless](https://github.com/OpenIntelWireless): itlwm / IntelBluetoothFirmware
- [USBToolBox](https://github.com/USBToolBox) and [corpnewt](https://github.com/corpnewt) toolchain

This EFI is provided for learning and reference only, at your own risk. Do not use someone else's SMBIOS values for activation or commercial purposes.
