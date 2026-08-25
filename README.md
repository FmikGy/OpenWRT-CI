# OpenWRT-CI - Link NN6000 V2 自用版

本仓库基于 [VIKINGYFY/OpenWRT-CI](https://github.com/VIKINGYFY/OpenWRT-CI) 修改，主要用于编译 **Link NN6000 V2** 的 ImmortalWrt 固件。

当前仅针对 NN6000 V2 做定制，保留上游 Qualcomm NSS 硬件加速，并同时构建：

- `IPQ60XX-WIFI-YES`：带 Wi-Fi 版本
- `IPQ60XX-WIFI-NO`：不带 Wi-Fi 版本

> 本仓库中的个人修改仅提交到本 fork，不修改上游仓库。

## 固件基础信息

| 项目 | 当前配置 |
| --- | --- |
| 设备 | Link NN6000 V2 |
| 平台 | Qualcomm IPQ60xx |
| 源码 | `VIKINGYFY/immortalwrt` |
| 分支 | `main` |
| 默认主题 | Aurora |
| 默认 LAN IP | `192.168.1.1` |
| DHCP 默认范围 | `192.168.1.100 - 192.168.1.249` |
| 默认主机名 | `Openwrt` |
| Wi-Fi SSID | `Openwrt` |
| Wi-Fi 密码 | `12345678` |
| LuCI 语言 | 简体中文 `zh_Hans` |
| SSH | OpenSSH |
| Dropbear | 禁用 |

## Wi-Fi / 无 Wi-Fi 双版本

`.github/workflows/QCA-ALL.yml` 使用 Matrix 同时构建两个配置：

```yaml
CONFIG: [IPQ60XX-WIFI-YES, IPQ60XX-WIFI-NO]
```

两个版本都只编译 **Link NN6000 V2**。

Wi-Fi 版保留设备正常的 ath11k / IPQ60xx Wi-Fi 支持；无 Wi-Fi 版关闭 ath / ath11k 相关驱动和固件，并使用项目现有的 Qualcomm 无 Wi-Fi DTS 调整逻辑。

两个版本使用不同的 `WRT_CONFIG`，Release Tag 和固件文件名也会分别区分，因此可以在同一次 Workflow 中同时发布。

## 当前主要插件与功能

### 网络与代理

- OpenClash
- SmartDNS + LuCI
- DDNS + LuCI
- WireGuard + LuCI 协议支持
- UPnP
- iperf3

### Web / 系统管理

- uHTTPd
- `uhttpd-mod-ubus`
- `luci-app-uhttpd`
- ttyd + `luci-app-ttyd`
- htop
- CPUFreq + `luci-app-cpufreq`
- Aurora 主题

### Docker

- `luci-app-dockerman`
- Docker 相关运行环境由 Dockerman 依赖自动带入

### 存储与文件共享

- DiskMan
- Samba4
- VSFTPD + `luci-app-vsftpd`
- block-mount
- USB Mass Storage
- UAS
- FAT / VFAT
- exFAT
- ext4
- NTFS3

### USB 支持

USB 核心与诊断：

- `kmod-usb-core`
- `kmod-usb2`
- `kmod-usb3`
- `usbutils`

USB 网络共享 / USB 网卡：

- RNDIS
- CDC Ethernet
- CDC-NCM
- ASIX
- AX88179
- Realtek RTL8152 / RTL8153 / RTL8156

适合常见 Android USB 网络共享、USB 千兆/2.5G 网卡以及 USB 存储设备。

### Node.js

- Node.js 22 LTS
- npm

Buildroot 配置使用：

```text
CONFIG_PACKAGE_node=y
CONFIG_PACKAGE_node-npm=y
CONFIG_NODEJS_22=y
```

当前 ImmortalWrt APK 中 Node.js 22 的运行时包可能显示为 `node127`，这是 Node ABI 包名，不需要把 Buildroot 配置改成 `CONFIG_PACKAGE_node127=y`。

### SSH

固件使用 OpenSSH：

- `openssh-server`
- `openssh-keygen`
- `openssh-sftp-server`

并禁用 Dropbear：

```text
CONFIG_PACKAGE_dropbear=n
```

OpenSSH 已配置允许 root 使用密码登录，但实际登录前仍需要给 root 设置密码。

## Qualcomm NSS

继续使用 `VIKINGYFY/immortalwrt` 对 qualcommax 平台提供的默认 Qualcomm NSS 加速组件，没有在私人配置中禁用 NSS。

包括 NSS Driver、ECM、DP、SSDK、Crypto 及相关 NSS Manager 等默认组件。

## 编译环境调整

已同步上游 `WRT-CORE.yml` 中与当前构建有关的 ncurses 依赖：

```bash
sudo -E apt -yqq install dos2unix libfuse-dev libncurses-dev libncurses6
```

其他与 NN6000 V2 无关的上游 QCB / IPQ53xx / IPQ95xx 等改动暂未整仓同步。

## 手动编译

进入仓库：

**Actions → QCA-ALL → Run workflow**

参数建议：

```text
Branch: main
PACKAGE: 留空
TEST: false
```

`TEST=false` 会正式编译固件，并同时启动：

```text
IPQ60XX-WIFI-YES
IPQ60XX-WIFI-NO
```

如果只想先检查最终 `.config`，可以把：

```text
TEST: true
```

这样可以先验证插件和目标设备配置，再正式编译。

## Release

正式编译完成后，Workflow 会创建 Release。

两个版本的 Tag 会分别包含：

```text
IPQ60XX-WIFI-YES-...
IPQ60XX-WIFI-NO-...
```

固件文件名也会包含：

```text
wifi-yes
wifi-no
```

因此两个版本不会互相覆盖。

## 重要配置文件

```text
.github/workflows/QCA-ALL.yml
    NN6000 V2 双版本 Workflow、IP、SSID、主机名等

.github/workflows/WRT-CORE.yml
    公共编译核心

Config/IPQ60XX-WIFI-YES.txt
    NN6000 V2 Wi-Fi 设备配置

Config/IPQ60XX-WIFI-NO.txt
    NN6000 V2 无 Wi-Fi 设备配置

Config/PRIVATE.txt
    私人插件和功能配置

Scripts/Settings.sh
    默认 IP、主机名、SSID、语言、OpenSSH 等定制逻辑

Scripts/Packages.sh
    Aurora、OpenClash、DiskMan 等第三方包更新逻辑
```

## 刷机注意事项

默认 LAN 地址为：

```text
192.168.1.1
```

默认 DHCP 范围由 dnsmasq 的 `start 100 / limit 150` 生成，因此在 `/24` LAN 下为：

```text
192.168.1.100 - 192.168.1.249
```

如果使用 sysupgrade 并选择 **保留配置**，旧设备原有的 `/etc/config/network`、Wi-Fi、主机名等配置可能继续保留；需要恢复默认值时可选择不保留旧配置或恢复出厂设置。

## 上游项目

本项目主要参考和使用以下项目：

- OpenWRT-CI：https://github.com/VIKINGYFY/OpenWRT-CI
- ImmortalWrt：https://github.com/immortalwrt/immortalwrt
- VIKINGYFY ImmortalWrt：https://github.com/VIKINGYFY/immortalwrt
- 自用 packages：https://github.com/VIKINGYFY/packages
- 本地编译工具：https://github.com/VIKINGYFY/OWRT-Tools

## U-Boot 相关资源

- Qualcomm / 沉心：https://github.com/chenxin527/uboot-qsdk12.5-build
- Qualcomm / 小猪：https://github.com/1980490718/u-boot-2016
- MediaTek U-Boot CI：https://github.com/VIKINGYFY/UBOOT-CI/releases

## 致谢

感谢 OpenWrt、ImmortalWrt、VIKINGYFY/OpenWRT-CI 以及相关插件和驱动项目的维护者。
