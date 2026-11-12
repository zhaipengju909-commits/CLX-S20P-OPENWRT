# CLX-S20P-OPENWRT

用 GitHub Actions 云编译的 **CLX S20P（超级网关 S20P，MT7986A）官方 OpenWrt 固件**。

> **练手项目。尚未验证能否开机，也不保证功能正常。刷机有变砖风险，后果自负。**
>
> *Practice project. Not verified to boot or to work on real hardware. Flashing can brick your device. Use at your own risk. No warranty.*

---

## 一、重要声明（请先读完）

1. **这是个人练手之作**，不是官方固件，与 OpenWrt、CLX、超级网关以及任何插件作者都没有关系。
2. **目前还没有在真机上验证过**：能不能开机、网口和 WiFi 是否正常、功能是否完整，都未知。已经做过的检查只有"能编译、镜像格式和设备树和原厂对得上"（见第四节），这不等于能用。
3. **刷机可能导致设备无法启动。** 刷之前请自己备份，并确认自己有办法恢复（U-Boot 网页恢复或串口）。没有恢复手段的话，请不要刷。
4. **不提供任何担保，也不提供售后和技术支持。** 因使用本项目造成的设备损坏、数据丢失、网络中断或其他损失，由使用者自己承担。
5. **请勿用于商业用途或转卖。** 固件中包含 GPL 等开源许可的软件，分发和使用需要遵守各自的许可证（见第六节）。
6. **版权方或插件作者如果希望修改署名、说明，或者下架相关内容，请在 Issues 里留言，我会尽快处理。**
7. 默认密码是公开的（见第二节），**上线前务必修改**，否则别人在局域网内就能登录你的路由器。

---

## 二、默认信息

| 项目 | 内容 |
| --- | --- |
| 管理地址 | http://192.168.2.1 |
| 用户名 | root |
| 密码 | openwrt（**首次登录后请立刻修改**） |
| 机型标识 | `clx,s20p` |
| 基础代码 | 官方 openwrt/openwrt，分支 `openwrt-25.12` |

> 注意：默认地址和密码是 2026-11-13 之后的版本才有的。更早的版本是官方默认的 192.168.1.1、无密码。每个 Release 的说明里都会写明基于哪个上游提交。

---

## 三、包含什么

- **官方原版 LuCI**，默认主题 bootstrap，中文界面。
- **全套内核模块**：`CONFIG_ALL_KMODS=y`，所有内核模块都编成了可安装的包，后面缺什么驱动都能装（打包在 Release 的 `kmod-and-packages.tar.zst` 里）。
- **额外插件**（官方源）：ttyd 终端、samba4 共享、wol 网络唤醒、sqm 限速、upnp、ddns、wireguard。
- **额外插件**（第三方源，编译时现拉）：iStore 商店（luci-app-store）、quickstart 首页、argon 和 aurora 两个主题、mini-diskmanager 磁盘管理。
- **没有预装代理类插件**（openclash、passwall 等），等能开机了再自己装。
- **没有的东西**：联发科 SDK 专用的 turboacc-mtk、eqos-mtk、mtwifi-cfg 不能用在官方内核上。硬件加速改用官方的"硬件流量卸载"，无线走官方 mt76 驱动。

---

## 四、已验证 / 未验证

**已验证（只代表"没发现明显矛盾"）：**

- 能编译通过，镜像校验值和发布的一致；
- 镜像里写的机型标识是 `clx,s20p`；
- 启动参数 `root=PARTLABEL=rootfs`、分区名、MAC 地址和 WiFi 校准数据的位置，和 CLX 原厂固件的设备树逐项对过，一致；
- 内核镜像格式、升级方式（sysupgrade-tar）、需要的软件包，和 CLX 原厂固件的构建配置一致；
- 选中的插件都进了软件清单。

**未验证（都需要真机）：**

- 能不能开机；
- 5 个 LAN 口和 WAN 口是否对应正确，2.5G 口是否正常；
- WiFi（2.4G 和 5G）是否正常、MAC 是否正确；
- 指示灯、按键、USB、M.2；
- 硬件流量卸载的实际效果和稳定性；
- 升级、保留配置等流程。

---

## 五、刷机前必读

1. **备份 factory 分区**（里面是 MAC 地址和 WiFi 校准数据，丢了很难恢复）。在现在的固件上 SSH 登录后，把对应分区用 `dd` 导出到电脑保存。分区号先用 `blkid` 确认，不要照抄别人的。
2. **确认恢复手段**：U-Boot 网页恢复或串口。没有就不要刷。
3. 用 LuCI 的 **系统 → 备份/刷写固件** 刷 `sysupgrade.bin`，**不要勾选"保留配置"**。
4. 刷完等几分钟，电脑网卡设成自动获取，访问 192.168.2.1。

---

## 六、来源与许可证

本项目的脚本只是把下面这些东西拼在一起，**没有修改它们的版权声明**：

| 内容 | 来源 | 许可证 |
| --- | --- | --- |
| 基础系统 | [openwrt/openwrt](https://github.com/openwrt/openwrt) | GPL-2.0 |
| 设备树 | [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt)（改名后使用） | GPL-2.0-or-later OR MIT（文件头声明） |
| luci-app-store | [linkease/istore](https://github.com/linkease/istore) | MIT |
| luci-app-mini-diskmanager | [4IceG/luci-app-mini-diskmanager](https://github.com/4IceG/luci-app-mini-diskmanager) | MIT |
| luci-theme-aurora、luci-app-aurora-config | [eamonxg](https://github.com/eamonxg) | Apache-2.0 |
| quickstart、luci-app-quickstart | [kenzok8/small-package](https://github.com/kenzok8/small-package) | 见其仓库及各包 Makefile（quickstart 的二进制是编译时从 linkease 的预编译发布页下载的） |
| luci-theme-argon、luci-app-argon-config | [sbwml/luci-theme-argon](https://github.com/sbwml/luci-theme-argon) | 见其仓库 |

其余软件包来自 OpenWrt 官方软件源，许可证各不相同。

**关于 GPL：** Release 里发布的固件包含 GPL 软件。对应的源码是：官方 OpenWrt 的指定提交（Release 里的 `upstream-commit.txt`）、本仓库的构建脚本，以及构建脚本里克隆的上述第三方仓库。以上只是说明，不是法律意见；如果要分发或商用，请自行核对各许可证的要求。

---

---

## 七、反馈

如果你真机试过，欢迎在 Issues 里告诉我结果：能不能开机、哪些功能有问题。
