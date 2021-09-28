---
layout: post
title: "PCI passthrough"
subtitle: "Ubuntu 20.04"
date: 2021-9-28
background: '/img/posts/06.jpg'
---
# 换源  
首先进行备份  
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```
换源
[https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

# 安装timeshift  
[https://github.com/teejee2008/timeshift](https://github.com/teejee2008/timeshift)
备份干净的os(comment: timeshift installed)
# 确认BIOS设置  
Advanced \ CPU config - SVM Module -> enable
Advanced \ AMD CBS - IOMMU -> enable
# 确认kernel version  
```
uname -r
```
确保版本不是5.1,5.2或5.3以及其子版本
# 安装QEMU，Libvirt，Virtualization manager  
```
sudo apt install qemu-kvm qemu-utils libvirt-daemon-system libvirt-clients bridge-utils virt-manager ovmf
```
备份(comment: QEMU，Libvirt，Virtualization manager installed)
# 启用 IOMMU
```
sudo nano /etc/default/grub
```
修改GRUB_CMDLINE_LINUX_DEFAULT
```
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt"
```
更新grub
```
sudo update-grub
```
重启
确认重启后IOMMU是否启动
```
dmesg |grep AMD-Vi
```
运行结果
```
[    1.938268] pci 0000:60:00.2: AMD-Vi: IOMMU performance counters supported
[    1.938306] pci 0000:40:00.2: AMD-Vi: IOMMU performance counters supported
[    1.938327] pci 0000:20:00.2: AMD-Vi: IOMMU performance counters supported
[    1.938348] pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported
[    1.941892] pci 0000:60:00.2: AMD-Vi: Found IOMMU cap 0x40
[    1.941894] AMD-Vi: Extended features (0x58f77ef22294ade): PPR X2APIC NX GT IA GA PC GA_vAPIC
[    1.941899] pci 0000:40:00.2: AMD-Vi: Found IOMMU cap 0x40
[    1.941901] AMD-Vi: Extended features (0x58f77ef22294ade): PPR X2APIC NX GT IA GA PC GA_vAPIC
[    1.941905] pci 0000:20:00.2: AMD-Vi: Found IOMMU cap 0x40
[    1.941906] AMD-Vi: Extended features (0x58f77ef22294ade): PPR X2APIC NX GT IA GA PC GA_vAPIC
[    1.941910] pci 0000:00:00.2: AMD-Vi: Found IOMMU cap 0x40
[    1.941911] AMD-Vi: Extended features (0x58f77ef22294ade): PPR X2APIC NX GT IA GA PC GA_vAPIC
[    1.941915] AMD-Vi: Interrupt remapping enabled
[    1.941916] AMD-Vi: Virtual APIC enabled
[    1.941917] AMD-Vi: X2APIC enabled
[    1.942195] AMD-Vi: Lazy IO/TLB flushing enabled

```
# 创建一个script，确认设备及其分组
```
touch test.sh
```
在test.sh 中写入
```
#!/bin/bash
# change the 999 if needed
shopt -s nullglob
for d in /sys/kernel/iommu_groups/{0..999}/devices/*; do
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done;
```
之后终端运行
```
chmod +x test.sh
./test.sh
```
运行结果为
```
IOMMU Group 0 60:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 1 60:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 2 60:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 3 60:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 4 60:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 5 60:05.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 6 60:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 7 60:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 8 60:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 9 60:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 10 61:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:2230] (rev a1)
IOMMU Group 10 61:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:1aef] (rev a1)
IOMMU Group 11 62:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Function [1022:148a]
IOMMU Group 12 63:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Reserved SPP [1022:1485]
IOMMU Group 13 40:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 14 40:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 15 40:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 16 40:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 17 40:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 18 40:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 19 40:05.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 20 40:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 21 40:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 22 40:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 23 40:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 24 41:00.0 VGA compatible controller [0300]: NVIDIA Corporation GK107GL [Quadro K420] [10de:0ff3] (rev a1)
IOMMU Group 24 41:00.1 Audio device [0403]: NVIDIA Corporation GK107 HDMI Audio Controller [10de:0e1b] (rev a1)
IOMMU Group 25 42:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:2230] (rev a1)
IOMMU Group 25 42:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:1aef] (rev a1)
IOMMU Group 26 43:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Function [1022:148a]
IOMMU Group 27 44:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Reserved SPP [1022:1485]
IOMMU Group 28 20:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 29 20:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 30 20:01.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 31 20:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 32 20:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 33 20:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 34 20:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 35 20:05.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 36 20:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 37 20:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 38 20:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 39 20:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 40 21:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd Device [144d:a80a]
IOMMU Group 41 22:00.0 Non-Volatile memory controller [0108]: Samsung Electronics Co Ltd Device [144d:a80a]
IOMMU Group 42 23:00.0 Ethernet controller [0200]: Intel Corporation I350 Gigabit Network Connection [8086:1521] (rev 01)
IOMMU Group 43 23:00.1 Ethernet controller [0200]: Intel Corporation I350 Gigabit Network Connection [8086:1521] (rev 01)
IOMMU Group 44 24:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Function [1022:148a]
IOMMU Group 45 25:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Reserved SPP [1022:1485]
IOMMU Group 46 25:00.1 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Cryptographic Coprocessor PSPCPP [1022:1486]
IOMMU Group 47 25:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Starship USB 3.0 Host Controller [1022:148c]
IOMMU Group 48 00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 49 00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 50 00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 51 00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 52 00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 53 00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 54 00:05.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 55 00:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 56 00:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 57 00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 58 00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 59 00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 61)
IOMMU Group 59 00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 60 00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 0 [1022:1490]
IOMMU Group 60 00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 1 [1022:1491]
IOMMU Group 60 00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 2 [1022:1492]
IOMMU Group 60 00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 3 [1022:1493]
IOMMU Group 60 00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 4 [1022:1494]
IOMMU Group 60 00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 5 [1022:1495]
IOMMU Group 60 00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 6 [1022:1496]
IOMMU Group 60 00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship Device 24; Function 7 [1022:1497]
IOMMU Group 61 01:00.0 Ethernet controller [0200]: Aquantia Corp. AQC107 NBase-T/IEEE 802.3bz Ethernet Controller [AQtion] [1d6a:07b1] (rev 02)
IOMMU Group 62 02:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Matisse Switch Upstream [1022:57ad]
IOMMU Group 63 03:08.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Matisse PCIe GPP Bridge [1022:57a4]
IOMMU Group 63 04:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Reserved SPP [1022:1485]
IOMMU Group 63 04:00.1 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Matisse USB 3.0 Host Controller [1022:149c]
IOMMU Group 63 04:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Matisse USB 3.0 Host Controller [1022:149c]
IOMMU Group 64 03:09.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Matisse PCIe GPP Bridge [1022:57a4]
IOMMU Group 64 05:00.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
IOMMU Group 65 03:0a.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Matisse PCIe GPP Bridge [1022:57a4]
IOMMU Group 65 06:00.0 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] [1022:7901] (rev 51)
IOMMU Group 66 07:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Function [1022:148a]
IOMMU Group 67 08:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Reserved SPP [1022:1485]
IOMMU Group 68 08:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Starship USB 3.0 Host Controller [1022:148c]

```
# 找到我们需要隔离的设备
```
IOMMU Group 25 42:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:2230] (rev a1)
IOMMU Group 25 42:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:1aef] (rev a1)
IOMMU Group 47 25:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Starship USB 3.0 Host Controller [1022:148c]

IOMMU Group 10 61:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:2230] (rev a1)
IOMMU Group 10 61:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:1aef] (rev a1)
IOMMU Group 68 08:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Starship USB 3.0 Host Controller [1022:148c]
```
# 通过设备id提供VFIO-pci driver
```
sudo vim /etc/default/grub
```
修改为
```
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt vifo-pci.ids=10de:2230"
```
更新
```
sudo update-grub
```
备份(VIFO-pci driver applyed)
重启