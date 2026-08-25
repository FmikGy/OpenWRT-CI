# OpenWRT-CI

本仓库基于 [VIKINGYFY/OpenWRT-CI](https://github.com/VIKINGYFY/OpenWRT-CI) 修改，用于编译 **Link NN6000 V2** 的 ImmortalWrt 固件。

源码：`VIKINGYFY/immortalwrt`  
分支：`main`

## 固件说明

同时编译并发布两个版本：

- `IPQ60XX-WIFI-YES`：Wi-Fi 版
- `IPQ60XX-WIFI-NO`：无 Wi-Fi 版

默认配置：

- LAN：`192.168.1.1`
- 主机名：`Openwrt`
- Wi-Fi 名称：`Openwrt`
- Wi-Fi 密码：`12345678`
- 主题：Aurora
- LuCI：简体中文
- 保留 Qualcomm NSS 硬件加速
- 使用 OpenSSH，禁用 Dropbear

主要包含 OpenClash、SmartDNS、Docker、DiskMan、Samba、UPnP、DDNS、WireGuard、FTP、Node.js 22、CPUFreq、uHTTPd、ttyd、iperf3、htop，以及常用 USB 存储和 USB 网络驱动。

## 编译

进入：

**Actions → QCA-ALL → Run workflow**

正式编译：

```text
Branch: main
PACKAGE: 留空
TEST: false
```

一次运行会同时编译 Wi-Fi 版和无 Wi-Fi 版，并分别发布 Release。

## 目录说明

- `.github/workflows`：CI 工作流
- `Config`：设备及插件配置
- `Scripts`：自定义脚本

## 上游项目

- OpenWRT-CI：https://github.com/VIKINGYFY/OpenWRT-CI
- ImmortalWrt：https://github.com/VIKINGYFY/immortalwrt
