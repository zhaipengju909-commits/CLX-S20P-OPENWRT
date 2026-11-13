# Full-cone NAT for nftables (not in official OpenWrt)

Official OpenWrt (openwrt-25.12) has no full-cone NAT support. This directory carries it, copied from
[immortalwrt/immortalwrt](https://github.com/immortalwrt/immortalwrt) (master, checked at 4765b657625e) and **adapted** to the
exact upstream versions that official openwrt-25.12 builds:

| Directory | What | Adapted |
| --- | --- | --- |
| `libnftnl/` | `fullcone` expression for libnftnl 1.3.1 | applies as-is (line offset only) |
| `nftables/` | `fullcone` statement for nftables 1.1.6 | scanner.l hunk re-written (official 1.1.6 has no `<*>` prefix on keywords) |
| `firewall4/` | `option fullcone` / `fullcone6` for firewall4 (commit b6e5157) | NAT action list re-written (official list has no `"npt"`) |
| `fullconenat-nft/` | kernel module `kmod-nft-fullcone` ([fullcone-nat-nftables/nft-fullcone](https://github.com/fullcone-nat-nftables/nft-fullcone) @ 07d93b6) + a kernel 6.12 build fix | unchanged |

The workflow dry-runs all patches against the exact upstream sources before building, so a version drift in
openwrt-25.12 stops the build early instead of producing a broken firewall.

Full-cone NAT is **off by default** (`option fullcone '0'`). Enable it with `option fullcone '1'` in the firewall `defaults` section.
If the kernel module cannot be loaded, firewall4 disables full-cone automatically (it logs a warning) and keeps working.

Licenses: the patches follow the licenses of the projects they modify (libnftnl and nftables: GPL-2.0-or-later;
firewall4: ISC; nft-fullcone: GPL-2.0-only). Original authors: Chion Tang, llccd, Syrone Wong (nft-fullcone);
the ImmortalWrt contributors (patches).
