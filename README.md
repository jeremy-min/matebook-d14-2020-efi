# Huawei MateBook D14 2020 Hackintosh EFI

一份针对 **Huawei MateBook D14 2020** 的 OpenCore EFI 配置，目标系统 **macOS 26 Tahoe**。基于十余轮实机调试整理，只保留已验证可用、且**不修改系统卷**的方案。

> ⚠️ **使用前必读**：本仓库的 `OC/config.plist` 已做身份脱敏（序列号、MLB、UUID、ROM 均为占位值）。**你必须替换成自己的值**，否则 iMessage / FaceTime / App Store 等依赖机器身份的服务无法使用。详见下方「首次使用」。

## 硬件配置

| 部件  | 型号                                               |
| --- | ------------------------------------------------ |
| 机型  | Huawei MateBook D14 2020                         |
| CPU | Intel Core i5-10210U（Comet Lake，4C8T）            |
| 核显  | Intel UHD Graphics 620（驱动内屏与副屏）                  |
| 独显  | NVIDIA GeForce MX250（Pascal GP108，**固件层断电，未启用**） |
| 内存  | 16 GB                                            |
| 无线  | Intel AX201（CNVi，WiFi 6 + 蓝牙）                    |
| 声卡  | Realtek ALC256                                   |
| 屏幕  | 内屏 2160×1440 3:2，HDMI 副屏                         |

## 已验证可用

| 功能                            | 方案                                                                                |
| ----------------------------- | --------------------------------------------------------------------------------- |
| 系统安装与启动                       | OpenCore，macOS 26 Tahoe                                                           |
| WiFi 原生菜单 + WiFi 6 (802.11ax) | `AirportItlwm-Tahoe.kext` + `itlwm_he=1`                                          |
| 蓝牙                            | `IntelBluetoothFirmware` + `BlueToolFixup` + `IntelBTPatcher`                     |
| 声卡（外放 / 耳机 / 内置麦克风）           | `AppleALC` + `SimpleHDA`，`layout-id=21`                                           |
| 亮度调节                          | `SSDT-PNLFCFL.aml`                                                                |
| 电池 / CPU 温度 / 风扇读数            | VirtualSMC 系列（`SMCBatteryManager`、`SMCProcessor`、`SMCHuaweiFan`、`SMCLightSensor`） |
| 触控板 / 键盘                      | `VoodooI2C` + `VoodooPS2Controller`                                               |
| HDMI 副屏                       | 核显输出，开机首次绿屏可能需重新插拔或合盖再开                                                           |
| USB                           | `USBInjectAll`（未做端口定制）                                                            |
| NVMe / 休眠 / 电源管理              | `NVMeFix`、`HibernationFixup`、原生 Comet Lake 电源管理                                   |

## 已知限制（不要重复踩坑）

- **8 个苹果**：接受现状，多次 SSDT / framebuffer 调整均无效。
- **副屏开机绿屏**：先绿后正常，属固件层，无法根治。
- **MX250 独显**：固件层断电、不在 PCI 总线上，驱动无解，已启用 `SSDT-dGPU-Off.aml` 关闭。Pascal 架构在 Tahoe 上无可用驱动。
- **指纹**：硬件与 TLS-PSK 协议层可通，但认证匹配分数长期低于阈值，已放弃并完整卸载用户态组件。
- **CPUFriend / 风扇调速 / 独显直连**：均不可行或无收益。
- **系统卷**：全程 sealed，**未做 OCLP root patch**。任何需要写系统卷的方案都不在本 EFI 范围内。
- **发布包范围**：仅保留 macOS 26 Tahoe 当前必需项；旧系统 WiFi 回退 kext、禁用指纹/SMC 文件、未使用主题与未引用工具未收录。

## 目录结构

```
OC/
├── ACPI/        # SSDT（EC/USBX、HPET、PLUG、PNLFCFL、PS2K、AWAC、ALS0、XOSI、dGPU-Off 等）
├── Drivers/     # OpenCore 驱动（OpenCanopy、OpenRuntime、HfsPlus、XhciDxe 等）
├── Kexts/       # 各类 kext（见下「致谢」）
├── Resources/   # 图标与主题资源
├── Tools/       # OpenShell、ResetSystem、CFGLock 等工具
├── config.plist # 主配置（已脱敏，需自行填写身份）
└── OpenCore.efi
```

## 首次使用

1. **填写机器身份**。用 ProperTree 或任何 plist 编辑器打开 `OC/config.plist`，在 `PlatformInfo → Generic` 中把以下占位值替换为你自己的（可用 [genxname](https://github.com/headkaze/gensn) 等工具生成）：
   - `SystemSerialNumber`：替换 `REPLACE_WITH_YOUR_SERIAL`
   - `MLB`：替换 `REPLACE_WITH_YOUR_MLB`
   - `SystemUUID`：替换全 0 UUID
   - `ROM`：填入你的网卡 MAC（6 字节十六进制）
2. `NVRAM → Add → 7C436110-... → bluetoothInternalControllerInfo` 已清空，如需要蓝牙控制器信息请自行填写。
3. 将整个 `OC` 文件夹复制到 EFI 分区的 `EFI/OC` 路径，确保 `Boot` 目录也在（本仓库不含 `Boot`，可从 OpenCore 官方包获取）。
4. 首次启动在 OpenCore 引导菜单选择 `ResetNvramEntry` 一次，然后正常安装 / 启动。
5. 如开机卡在引导，检查 FAT32 分区 dirty bit：`sudo diskutil repairVolume disk0s1`。

## 已注入 kext（致谢）

本 EFI 使用了以下社区项目，版权归各自作者所有：

- [Lilu](https://github.com/acidanthera/Lilu) — 所有插件的基础
- [WhateverGreen](https://github.com/acidanthera/WhateverGreen) — 显卡补丁
- [AppleALC](https://github.com/acidanthera/AppleALC) + [SimpleHDA](https://github.com/laobamac/SimpleHDA) — 声卡
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC) 及 SMC 传感器子 kext
- [NVMeFix](https://github.com/acidanthera/NVMeFix)、[RestrictEvents](https://github.com/acidanthera/RestrictEvents)、[HibernationFixup](https://github.com/acidanthera/HibernationFixup)
- [AirportItlwm-Tahoe](https://github.com/laobamac/itlwm)（基于 [OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm)）— Intel WiFi
- [IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) + [BlueToolFixup](https://github.com/acidanthera/BlueToolFixup) + [IntelBTPatcher](https://github.com/Sherryha6/IntelBTPatcher) — 蓝牙
- [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) + [VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2Controller) — 触控板与键盘
- [USBInjectAll](https://github.com/externalist/OS-X-USB-Inject-All) — USB
- 其余 kext / EFI 驱动来自 [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) 及对应开源项目

## 免责声明

本项目仅供学习与研究使用，按「现状」提供，不保证在任何其他机器或系统版本上可用。使用黑苹果及第三方驱动存在系统不稳定、数据丢失的风险，请自行备份。所有第三方 kext / 驱动的知识产权归原作者所有，本项目不包含任何商业软件或受版权保护的系统组件。

本项目与 Huawei、Apple、NVIDIA、Intel 等公司无任何关联。
