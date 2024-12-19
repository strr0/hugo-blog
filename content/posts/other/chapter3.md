---
title: "基于 OpenCore 引导的黑苹果安装"
date: 2023-10-01T10:00:00+08:00
categories: ["other"]
draft: false
---

## 介绍

本教程基于 [OpenCore 官方教程](https://dortania.github.io/OpenCore-Install-Guide)，开始之前需要[确认你的硬件是否支持](https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html)，其次确认你是否真的需要黑苹果，最后确保你有足够的时间和耐心。

## 1 安装

### 1.1 准备

- 4G 及以上的 U 盘
- [OpenCorePkg 镜像下载工具](https://github.com/acidanthera/OpenCorePkg/releases)（基于python）
- [Rufus格式化工具](https://rufus.ie/)（非必要）

#### 1.1.1 下载 macOS

命令行进入 OpenCorePkg，并进入镜像下载工具 macrecovery 目录
```
cd ./Utilities/macrecovery
```
下载需要安装的macOS版本
```
# Lion (10.7):
python3 macrecovery.py -b Mac-2E6FAB96566FE58C -m 00000000000F25Y00 download
python3 macrecovery.py -b Mac-C3EC7CD22292981F -m 00000000000F0HM00 download

# Mountain Lion (10.8):
python3 macrecovery.py -b Mac-7DF2A3B5E5D671ED -m 00000000000F65100 download

# Mavericks (10.9):
python3 macrecovery.py -b Mac-F60DEB81FF30ACF6 -m 00000000000FNN100 download

# Yosemite (10.10):
python3 macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000GDVW00 download

# El Capitan (10.11):
python3 macrecovery.py -b Mac-FFE5EF870D7BA81A -m 00000000000GQRX00 download

# Sierra (10.12):
python3 macrecovery.py -b Mac-77F17D7DA9285301 -m 00000000000J0DX00 download

# High Sierra (10.13)
python3 macrecovery.py -b Mac-7BA5B2D9E42DDD94 -m 00000000000J80300 download
python3 macrecovery.py -b Mac-BE088AF8C5EB4FA2 -m 00000000000J80300 download

# Mojave (10.14)
python3 macrecovery.py -b Mac-7BA5B2DFE22DDD8C -m 00000000000KXPG00 download

# Catalina (10.15)
python3 macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download

# Big Sur (11)
python3 macrecovery.py -b Mac-42FD25EABCABB274 -m 00000000000000000 download

# Monterey (12)
python3 macrecovery.py -b Mac-FFE5EF870D7BA81A -m 00000000000000000 download

# Latest version
# ie. Ventura (13)
python3 macrecovery.py -b Mac-4B682C642B45593E -m 00000000000000000 download
```

#### 1.1.2 制作启动盘

使用磁盘工具（此电脑 -> 管理 -> 磁盘管理）格式化 U 盘为 FAT32 格式（或者 Rufus 工具）。

{{< image src="images/other/chapter3/DiskManagement.jpg" width="auto" height="auto" >}}

在 U 盘根目录创建 com.apple.recovery.boot 目录，并将下载的镜像文件 BaseSystem 放到该目录下。

{{< image src="images/other/chapter3/com-recovery.png" width="auto" height="auto" >}}

打开下载的 OpenCorePkg 目录

{{< image src="images/other/chapter3/base-oc-folder.png" width="auto" height="auto" >}}

从 IA32 （32位）或 X64 （64位）目录选择合适的 EFI 放入 U 盘根目录，最终如下：

{{< image src="images/other/chapter3/com-efi-done.png" width="auto" height="auto" >}}

### 1.2 基本配置文件

保留基本配置文件，删除多余配置文件，最终如下：

{{< image src="images/other/chapter3/clean-efi.png" width="auto" height="auto" >}}

### 1.3 收集文件

#### 通用驱动

- [HfsPlus.efi](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi)（必要）
- OpenRuntime.efi（必要，已存在基本配置文件中）

#### 通用kexts

- [Lilu](https://github.com/acidanthera/Lilu/releases)
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases)

#### VirtualSMC可选插件

- SMCProcessor.kext （cpu温度）
- SMCSuperIO.kext （风扇转速）
- SMCLightSensor.kext （光传感器）
- SMCBatteryManager.kext （电池读数）

#### 图形化

- [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)（必要）

#### 音频

- [AppleALC](https://github.com/acidanthera/AppleALC/releases)
  [有线网络](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#ethernet)（自行选择合适驱动）
- [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)

#### USB映射（可在系统安装完成后配置）

#### [无线网络及蓝牙](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#wifi-and-bluetooth)（自行选择合适驱动）

- [AirportItlwm](https://github.com/OpenIntelWireless/itlwm/releases)
- [IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases)

#### [其他](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#extras)

- [NVMeFix](https://github.com/acidanthera/NVMeFix/releases)

#### [笔记本](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#laptop-input)（自行选择合适驱动）

- [VoodooPS2](https://github.com/acidanthera/VoodooPS2/releases)
- [VoodooRMI](https://github.com/VoodooSMBus/VoodooRMI/releases)
- [ECEnabler](https://github.com/1Revenger1/ECEnabler/releases)（电池问题）
- [BrightnessKeys](https://github.com/acidanthera/BrightnessKeys/releases)

#### [SSDTs](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#ssdts)（根据实际情况自行选择）

- [SSDT-PLUG](https://dortania.github.io/Getting-Started-With-ACPI/Universal/plug.html)
- [SSDT-EC-USBX](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html)
- [SSDT-PNLF](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/backlight.html)
- [SSDT-GPI0](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/trackpad.html)（可用 SSDT-XOSI 代替）

### 1.4 [config.plist 配置](https://dortania.github.io/OpenCore-Install-Guide/config.plist/#selecting-your-platform)（根据实际情况配置）

#### ACPI -> Patch 添加如下

| Comment | String  | Change _OSI to XOSI |
| :------ | :------ | :------------------ |
| Enabled | Boolean | YES                 |
| Count   | Number  | 0                   |
| Limit   | Number  | 0                   |
| Find    | Data    | `5f4f5349`          |
| Replace | Data    | `584f5349`          |

#### DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0) 添加如下

| Key                      | Type | Value      |
| :----------------------- | :--- | :--------- |
| AAPL,ig-platform-id      | Data | `00001B59` |
| framebuffer-patch-enable | Data | `01000000` |
| framebuffer-stolenmem    | Data | `00003001` |
| framebuffer-fbmem        | Data | `00009000` |

#### Kernel -> Quirks 修改如下

| Quirk                   | Enabled | Comment                                          |
| :---------------------- | :------ | :----------------------------------------------- |
| AppleXcpmCfgLock        | YES     | Not needed if `CFG-Lock` is disabled in the BIOS |
| DisableIoMapper         | YES     | Not needed if `VT-D` is disabled in the BIOS     |
| LapicKernelPanic        | NO      | HP Machines will require this quirk              |
| PanicNoKextDump         | YES     |                                                  |
| PowerTimeoutKernelPanic | YES     |                                                  |
| XhciPortLimit           | YES     | Disable if running macOS 11.3+                   |

#### Misc -> Boot 修改如下

| Quirk         | Enabled | Comment                                                      |
| :------------ | :------ | :----------------------------------------------------------- |
| HideAuxiliary | YES     | Press space to show macOS recovery and other auxiliary entries |

#### Misc -> Debug 修改如下

| Quirk           | Enabled |
| :-------------- | :------ |
| AppleDebug      | YES     |
| ApplePanic      | YES     |
| DisableWatchDog | YES     |
| Target          | 67      |

#### Misc -> Security 修改如下

| Quirk                | Enabled  | Comment                                                      |
| :------------------- | :------- | :----------------------------------------------------------- |
| AllowSetDefault      | YES      |                                                              |
| BlacklistAppleUpdate | YES      |                                                              |
| ScanPolicy           | 0        |                                                              |
| SecureBootModel      | Default  | Leave this as `Default` for OpenCore to automatically set the correct value corresponding to your SMBIOS. The next page goes into more detail about this setting. |
| Vault                | Optional | This is a word, it is not optional to omit this setting. You will regret it if you don't set it to Optional, note that it is case-sensitive |

#### NVRAM -> Add -> 7C436110... 修改如下

| Key            | Type   | Value                                                                     |
| :------------- | :----- | :------------------------------------------------------------------------ |
| boot-args      | String | `-v keepsyms=1 debug=0x100 alcid=1 -wegnoegpu -igfxnotelemetryload` |
| prev-lang:kbd  | String | `en-US:0`                                                                 |

#### PlatformInfo -> Generic 修改如下（使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 生成序号）

| Key                | Type   | Value           |
| :----------------- | :----- | :-------------- |
| SystemProductName  | String | 生成Type         |
| SystemSerialNumber | String | 生成Serial       |
| MLB                | String | 生成Board Serial |
| SystemUUID         | String | 生成SmUUID       |
| ROM                | DATA   | Apple ROM       |

#### UEFI -> Quirks 修改如下

| Quirk               | Enabled | Comment                          |
| :------------------ | :------ | :------------------------------- |
| ReleaseUsbOwnership | YES     |                                  |
| UnblockFsConnect    | NO      | Needed mainly by HP motherboards |

### 1.5 配置完成后可执行安装

## 2 安装后配置

### 2.1 [USBMap](https://github.com/corpnewt/USBMap) 配置

1. 运行 USBMap.command，选择 D. Discover Ports 发现端口

2. 选择 K. Create USBMapDummy.kext 生成虚拟注入器

3. 将生成的虚拟注入器添加到 EFI/OC/Kexts 目录及 config.plist -> Kernel -> Add 配置中

4. 重启Mac

5. 运行 USBMap.command，选择 D. Discover Ports 发现端口，并将 USB 2.0 、USB 3.0 设备插入每个端口中

6. 使用 USBMapInjectorEdit.command 选择 EFI 中的 USBMapDummy.kext，关闭多余的端口

7. 重启Mac

8. 重复步骤 5

9. 选择 P. Edit & Create USBMap.kext 修改端口类型，并生成最终 USBMap.kext

10. 用 USBMap.kext 替换 USBMapDummy.kext，并修改 config.plist

### 2.2 引导 UI 页面配置

1. 添加 [OpenCanopy.efi](https://github.com/acidanthera/OpenCorePkg/releases) 到 EFI/OC/Drivers 和 config.plist -> UEFI -> Drivers 配置中

2. 添加 [OcBinaryData](https://github.com/acidanthera/OcBinaryData) 的 Resources 到 EFI/OC/Resources

3. 修改 Misc -> Boot 配置

| Key              | Type   | Value             |
| :--------------- | :----- | :---------------- |
| PickerMode       | String | External          |
| PickerAttributes | String | 17                |
| PickerVariant    | String | Acidanthera\Syrah |

4. 若出现引导页面黑屏（可以盲敲选择），尝试修改 config.plist 的 ProvideConsoleGOP 为 false

### 2.3 添加 OpenCore 引导到 bios 中

1. 修改 Misc -> Boot -> LauncherOption 为 Full （系微固件则为 Short ）

2. 修改 UEFI -> Quirks -> RequestBootVarRouting 为 True

3. 首次使用 OpenCore 引导（从EFI文件中选择）后，引导将添加至 bios 中

### 2.4 多系统配置（Linux）

1. 安装 Linux

2. 添加 OpenLinuxBoot.efi 和 Ext4Dxe.efi 到 Drivers 和 config.plist 配置中

3. 确保 RequestBootVarRouting 和 LauncherOption 开启

4. 重启即可看到 Linux 启动项