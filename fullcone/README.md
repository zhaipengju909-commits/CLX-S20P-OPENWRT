# Full-cone NAT for nftables (not in official OpenWrt)

Official OpenWrt (openwrt-25.12) has no full-cone NAT support. This directory carries it, copied **unmodified** from the
`openwrt-25.12` branch of [immortalwrt/immortalwrt](https://github.com/immortalwrt/immortalwrt). That branch builds the same
upstream versions as official openwrt-25.12 (kernel 6.12, libnftnl 1.3.1, nftables 1.1.6, firewall4 b6e5157).
(Do not use the patches from ImmortalWrt master: it has moved to libnftnl 1.3.2 and its patch does not build against 1.3.1.)

| Directory | What |
| --- | --- |
| `libnftnl/` | `fullcone` expression for libnftnl 1.3.1. The workflow also enables `PKG_FIXUP:=autoreconf` for the package, because the patch edits `src/Makefile.am` (ImmortalWrt's libnftnl Makefile does the same) |
| `nftables/` | `fullcone` statement for nftables 1.1.6 |
| `firewall4/` | `option fullcone` / `fullcone6` for firewall4 (commit b6e5157) |
| `fullconenat-nft/` | kernel module `kmod-nft-fullcone` ([fullcone-nat-nftables/nft-fullcone](https://github.com/fullcone-nat-nftables/nft-fullcone) @ 07d93b6) + a kernel 6.12 build fix |

The workflow dry-runs all patches against the exact upstream sources before building, so a version drift in
openwrt-25.12 stops the build early instead of producing a broken firewall.

The ImmortalWrt firewall4 patch changes the default `/etc/config/firewall` (`defaults` section) to: `flow_offloading 1`, `flow_offloading_hw 1`, `fullcone 1`, `fullcone6 0` (official firewall4 has none of these). So **full-cone NAT and flow offloading (software and hardware) are ON by default**; set the option to `0` to turn each off.
If the kernel module cannot be loaded, firewall4 disables full-cone automatically (it logs a warning) and keeps working.

Licenses: the patches follow the licenses of the projects they modify (libnftnl and nftables: GPL-2.0-or-later;
firewall4: ISC; nft-fullcone: GPL-2.0-only). Original authors: Chion Tang, llccd, Syrone Wong (nft-fullcone);
the ImmortalWrt contributors (patches).

## Kernel patch (`kernel/`)
`kernel/952-net-conntrack-events-support-multiple-registrant.patch` (ImmortalWrt openwrt-25.12 `target/linux/generic/hack-6.12/`, unmodified)
adds the optional `CONFIG_NF_CONNTRACK_CHAIN_EVENTS`, which `kmod-nft-fullcone` selects. Without it, official kernel 6.12 has a single conntrack-event
notifier slot: `nft_fullcone` takes it, and a later `nf_conntrack_netlink` (pulled in by e.g. `dnsmasq-full`) overwrites it with a kernel WARNING,
so full-cone loses its connection-destroy events. The workflow dry-runs this patch against the exact kernel source before building.
