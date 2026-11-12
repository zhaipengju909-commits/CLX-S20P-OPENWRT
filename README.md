# CLX-S20P-OPENWRT

用 GitHub Actions 云编译的 **CLX S20P（超级网关 S20P，MT7986A）官方 OpenWrt 固件**。

> **练手项目。尚未验证能否开机，也不保证功能正常。刷机有变砖风险，后果自负。**
>
> *Practice project. Not verified to boot or to work on real hardware. Flashing can brick your device. Use at your own risk. No warranty.*

---

## 一、重要声明（请先读完）

1. **这是个人练手之作**，不是官方固件，与 OpenWrt、CLX、超级网关以及任何插件作者都没有关系。
2. **目前还没有在真机上验证过**：能不能开机、网口和 WiFi 是否正常、功能是否完整，都未知。已经做过的检查只有"能编译、镜像格式和设备树和原厂对得上"（见第五节），这不等于能用。
3. **刷机可能导致设备无法启动。** 刷之前请自己备份，并确认自己有办法恢复（U-Boot 网页恢复或串口）。没有恢复手段的话，请不要刷。
4. **不提供任何担保，也不提供售后和技术支持。** 因使用本项目造成的设备损坏、数据丢失、网络中断或其他损失，由使用者自己承担。
5. **请勿用于商业用途或转卖。** 固件中包含 GPL 等开源许可的软件，分发和使用需要遵守各自的许可证（见第七节）。
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

## 四、怎么编译

1. 进入 **Actions** 页，选 **Build CLX S20P (official OpenWrt)**，点 **Run workflow**。
2. `branch` 默认 `openwrt-25.12`。勾选 `config_only` 只做配置检查，几分钟出结果，不编固件，用来快速试错。
3. 完整编译大约 1.5 到 2.5 小时。成功后固件会发布到 **Releases**，里面有 `sysupgrade.bin`、软件清单、校验值、配置文件、上游提交号。

设备支持是怎么加的：构建时对官方 OpenWrt 打补丁（脚本在 `.github/workflows/build.yml` 的 "Add S20P support" 一步），内容是：

- 从 VIKINGYFY/immortalwrt 的固定提交取设备树，把名字里的 `supergateway` 换成 `clx`，机型标识保持 `clx,s20p`，这样能直接从现在的 CLX 固件升级过来；
- 在官方 `filogic.mk`、`platform.sh`、`02_network`、`01_leds` 里加上这个机型。

---

## 五、已验证 / 未验证

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

## 六、刷机前必读

1. **备份 factory 分区**（里面是 MAC 地址和 WiFi 校准数据，丢了很难恢复）。在现在的固件上 SSH 登录后，把对应分区用 `dd` 导出到电脑保存。分区号先用 `blkid` 确认，不要照抄别人的。
2. **确认恢复手段**：U-Boot 网页恢复或串口。没有就不要刷。
3. 用 LuCI 的 **系统 → 备份/刷写固件** 刷 `sysupgrade.bin`，**不要勾选"保留配置"**。
4. 刷完等几分钟，电脑网卡设成自动获取，访问 192.168.2.1。

---

## 七、来源与许可证

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

## 八、编译失败复盘与改进

这次共有 2 次编译失败（都发生在开始后约 5 分钟内的"生成配置"那一步），之后的完整编译成功了 2 次。失败不是源码问题，**是我写的配置有错**。

| 问题 | 原因 | 后果 | 怎么发现的 |
| --- | --- | --- | --- |
| 编译失败 2 次 | 设备选项的名字写错了：应该是 `CONFIG_TARGET_mediatek_filogic_DEVICE_clx_s20p`，我写成了 `CONFIG_TARGET_DEVICE_mediatek_filogic_DEVICE_clx_s20p` | 如果没有检查，会静默编出另一个机型（openwrt_one）的固件，刷进去可能变砖 | 配置后的检查没通过；第二次加了诊断输出，日志里看到系统改选了别的机型 |
| iStore 商店被悄悄丢掉 | 它依赖"tar 带 xz 支持时必须同时装 xz-utils"，我没选 xz-utils，配置阶段会静默删掉它，不报错 | 固件里少了商店 | 加了"插件是否被丢弃"的检查，日志里显示 `PLUGIN DROPPED` |

**根本原因：** 云端编译环境我这边没有，只能靠推送、等结果，所以第一次的配置没法先试。而且我凭记忆写了选项名，没有先对照源码核实。

**以后怎么做：**

1. **先核对，再提交**：设备、软件包的选项名先对照上游源码，确认存在，再写进配置。
2. **先快后慢**：任何改动先跑 `config_only`（几分钟），确认配置没问题，再跑完整编译（1.5 小时以上）。
3. **检查要能说清原因**：不只写"失败"，要把关键选项、被丢弃的插件、缺失的依赖直接打印出来。
4. **关键项必须硬检查**：目标机型不对就立刻中止，不再往下编。第三方插件掉了只告警，不影响出固件。
5. **成品必验**：编完下载镜像，核对校验值、机型标识、启动参数、设备树；默认 IP 和密码在构建过程中再检查一遍。
6. **固定版本**：Release 里记录上游提交号，出问题能复现。
7. **说明写清楚**：每个 Release 都带风险说明和默认账号，不让使用者靠猜。

---

## 九、已知限制和后续计划

- 真机验证是最大的缺口，有人验证后请到 Issues 反馈（能不能开机、哪些功能有问题）。
- 第三方插件依赖外部仓库的当前版本，上游改动可能导致编译失败。
- 计划：真机能开机后，再补代理类插件、调整默认主题、完善无线和硬件加速的配置。
