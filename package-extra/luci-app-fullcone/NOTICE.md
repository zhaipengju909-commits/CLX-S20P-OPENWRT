# luci-app-fullcone

This directory is a copy of `package/luci-app-fullcone` from
https://github.com/akorshun/fullcone-openwrt (Apache-2.0, see the SPDX header in `Makefile`),
taken unmodified, plus one added file: `po/zh_Hans/fullcone.po` (Simplified Chinese translation).

It is only the LuCI page (Network → Firewall → FullCone NAT). The firewall4 / nftables / libnftnl / kernel-module
parts of full-cone NAT in this project come from `../../fullcone/` (see its README).
