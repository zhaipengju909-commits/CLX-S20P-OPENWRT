# CLX-S20P-OPENWRT

用 GitHub Actions 云编译的 **CLX S20P（超级网关 S20P，MT7986A）官方 OpenWrt 固件**。

> **练手项目。已在 1 台 CLX S20P 真机上验证:能开机、能拨号上网、有 IPv6、2.4G/5G Wi-Fi 能起 AP(见第四节)。仍可能有没发现的问题。刷机有变砖风险,后果自负。**
>
> *Practice project. Verified on ONE CLX S20P unit only: boots, PPPoE/IPv6, Wi-Fi AP on both bands (see section 4). Other problems may exist. Flashing can brick your device. Use at your own risk. No warranty.*

---

## 一、重要声明（请先读完）

1. **这是个人练手之作**，不是官方固件，与 OpenWrt、CLX、超级网关以及任何插件作者都没有关系。
2. **只在 1 台真机上验证过**(详见第四节)：其他机器、其他刷机来源、长时间稳定性都没有验证。
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

- **官方原版 LuCI**，默认主题 aurora。自 b12 起已内置简体中文语言包（b18 真机实测 12 个 `luci-i18n-*-zh-cn`）；`s20p-20261113-0049` 及更早版本界面为英文。
- **全套内核模块**：`CONFIG_ALL_KMODS=y`，所有内核模块都编成了可安装的包，后面缺什么驱动都能装（打包在 Release 的 `*-packages.tar.zst` 里，文件名前缀与构建名一致）。
- **额外插件**（官方源）：ttyd 终端、samba4 共享、upnp、ddns、wireguard、`luci-compat`（兼容旧版 Lua LuCI 应用）。wol 与 sqm 已从镜像移除。
- **额外插件**（第三方源，编译时现拉）：iStore 商店（luci-app-store）、quickstart 首页、aurora 主题、mini-diskmanager 磁盘管理。
- **没有预装代理类插件**（openclash、passwall 等），等能开机了再自己装。
- **全锥型 NAT**：官方 OpenWrt 主线不含 fullcone，本项目通过 `kmod-nft-fullcone` + firewall4 补丁提供 IPv4 全锥形（`luci-app-fullcone` 可管理）。镜像默认不开，刷机后由恢复脚本设 `firewall.@defaults[0].fullcone=1`、`fullcone6=0`。b18 实测：同一本地端口对多家 STUN 服务器映射一致，换 IP + 换端口的回包可进入内网。
- **没有的东西**：联发科 SDK 专用的 turboacc-mtk、eqos-mtk、mtwifi-cfg 不能用在官方内核上；硬件加速改用官方主线自带的"硬件流量卸载"（b18 实测 PPE 流表绑定生效、CPU 占用低），无线走官方 mt76 驱动。

---

## 四、已验证 / 未验证

**已在 1 台真机上验证(2026-11-13,构建 `s20p-20261113-0049`,该机之前运行的是 CLX 系列第三方固件):**

- 能开机,机型标识 `clx,s20p`,引导参数、分区、MAC 地址(来自 factory 分区)和原固件一致；
- PPPoE 拨号上网正常,能获取 IPv6 全局地址；WAN 口(eth1)2.5G 正常；
- 共 **6 个 LAN 口**(lan1~lan6),其中 **lan6 能协商 2.5G**,其余为千兆。实测 lan3~lan6 接设备时千兆正常,lan1、lan2 没接设备,没测；
- Wi-Fi:2.4G 和 5G 都能起 AP,MAC 正确,能扫描到周围的 AP。**只验证了能起 AP 和扫描**,没有做吞吐和长时间稳定性测试。出厂默认两个 Wi-Fi 接口都是关闭的；
- 状态灯和按键的配置已生效,USB 主控被识别(没有按键实测,没有接 USB 设备)；
- 闪存分区布局保持不变,eMMC 健康信息正常；
- 硬件流量卸载:能启用,联发科流量引擎里能看到已绑定的 PPPoE 加 NAT 的流表,CPU 占用低。**吞吐提升没有做过对照测量**；
- 在原固件上用 `sysupgrade -T` 检验过本镜像,通过。

**还没验证:**

- 按键的实际动作、USB 外接设备、M.2/PCIe(没有接设备时 PCIe 控制器探测超时,接了固态会怎样未知)；
- 硬件流量卸载的实际吞吐和长期稳定性；
- 保留配置的升级流程、从本固件升级到下一个版本；
- Wi-Fi 的吞吐和稳定性；
- 在其他机器、其他分区布局、原厂系统上刷机。

**已知问题和解决办法(`s20p-20261113-0049`,仅适用于该历史版本;当前状态见下节):**

1. 界面是英文:这个版本漏装了简体中文语言包。**b12 起已修复**。
2. iStore 里一些第三方应用(例如 homebox)打开时报 `module 'luci.cbi' not found`:这类应用用的是老的 Lua 界面框架,需要 `luci-compat`。0049 上可执行 `apk add luci-compat`(官方软件源里有);**b12 起已编入镜像**。
3. 官方软件源里**没有适配本固件内核的内核模块**,需要时使用本 Release 里的 `*-packages.tar.zst`。解压后把**整个目录**复制到路由器的 `/tmp/pk`(不能只拷一两个文件);推荐把本地源写进 `/etc/apk/repositories.d/zz-local.list` 后执行 `apk update`,再 `apk add <包名>`。也可直接 `apk add --repository /tmp/pk/packages.adb <包名>`(个别环境会因已安装包的索引报 `no such package`,此时改用前一种方式)。软件源索引的签名可以被固件直接信任,不需要 `--allow-untrusted`。**注**:b18 起常用内核模块(见下节)已编入镜像,本条主要用于其他驱动。
4. **本固件不带在线更新功能**:没有安装 `attendedsysupgrade` / `owut`,"备份/刷写固件"页面只能手动上传镜像。升级请到本仓库的 Releases 页面手动下载新版本。官方 OpenWrt 的源码里没有 `clx_s20p` 这个机型,所以**请不要用官方的在线固件升级(attendedsysupgrade)来升级本固件**,会发生什么没有验证过。
5. **"软件包"页面的"更新列表"查的是官方软件源**,和本固件的版本无关。官方源里没有适配本固件内核的内核模块,**请不要点"全部升级"**,否则基础包可能和本固件不匹配(没有验证过具体后果);只安装你需要的单个软件包,需要内核模块时按第 3 条使用 Release 里的同批软件包。

**当前版本状态更新(b18 `r0-3c132d0`,内核 6.12.116,2026-11-14 真机实测):**

- 0049 的第 1、2 条问题已修复:内置 12 个简体中文语言包与 `luci-compat`;wol、sqm 及对应自启项已从镜像移除。
- 代理插件所需内核模块 `kmod-tun`、`kmod-nft-tproxy`、`kmod-inet-diag`、`kmod-netlink-diag`、`kmod-nf-conntrack-netlink`,以及 `dnsmasq-full`、`coreutils-nohup`、`openssh-sftp-server`、`kmod-nft-fullcone` 已编入镜像,刷机后无需再为这些包挂本地源。**其他**内核模块仍按上文第 3 条使用 Release 里的同批软件包。
- 内核已带 conntrack 事件链补丁,`dmesg` 无 `nf_conntrack_register_notifier` WARNING。
- 出厂两个无线接口均为 disabled;时区由恢复脚本设为 `Asia/Shanghai` / `CST-8`;主机名仍为默认 `OpenWrt`。
- IPv4 全锥形 NAT 已可用,实测行为为 NAT1(Full Cone)。浏览器类 NAT 检测工具显示的 NAT3 属于客户端(WebRTC/浏览器)不回应换 IP 探测所致,已由 WAN/LAN 同步抓包证实与固件无关,请勿据此修改 NAT 配置。
- 官方主线硬件流量卸载已实测生效,但"开/关"吞吐对照测量仍未完成。
- 上文第 4、5 条**仍然适用**:本固件不带在线更新功能,且不要点"全部升级"。

---

## 五、刷机前必读

0. **分区要求**:本固件的升级流程要求闪存分区表里有名为 `kernel` 和 `rootfs` 的分区(CLX 系列第三方固件的分区就是这样)。**只在这样的机器上验证过,没有在原厂系统上验证过。**
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

其余软件包来自 OpenWrt 官方软件源，许可证各不相同。

**关于 GPL：** Release 里发布的固件包含 GPL 软件。对应的源码是：官方 OpenWrt 的指定提交（Release 里的 `upstream-commit.txt`）、本仓库的构建脚本，以及构建脚本里克隆的上述第三方仓库。以上只是说明，不是法律意见；如果要分发或商用，请自行核对各许可证的要求。

---

---

## 七、反馈

如果你真机试过，欢迎在 Issues 里告诉我结果：能不能开机、哪些功能有问题。
