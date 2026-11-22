<p align="center"><img src="https://raw.githubusercontent.com/kenzok8/kenzok8/main/screenshot/usque/usque-logo.png" width="88" height="88" alt="Usque"></p>
<h1 align="center">Usque for OpenWrt</h1>
<p align="center">See your WARP connection clearly, know where your traffic goes.</p>

<p align="center">English · <a href="README.md">中文</a></p>

<p align="center">
<a href="https://github.com/kenzok8/openwrt-usque/releases"><img alt="Release" src="https://img.shields.io/github/v/release/kenzok8/openwrt-usque?include_prereleases&color=a74414"></a>
<a href="https://github.com/kenzok8/openwrt-usque/releases"><img alt="Downloads" src="https://img.shields.io/github/downloads/kenzok8/openwrt-usque/total"></a>
<a href="https://github.com/kenzok8/openwrt-usque/commits/main"><img alt="Last commit" src="https://img.shields.io/github/last-commit/kenzok8/openwrt-usque"></a>
<a href="https://github.com/kenzok8/openwrt-usque/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/kenzok8/openwrt-usque"></a>
<a href="https://github.com/kenzok8/openwrt-usque/issues?q=is%3Aissue+is%3Aclosed"><img alt="Closed issues" src="https://img.shields.io/github/issues-closed/kenzok8/openwrt-usque"></a>
<a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/license-GPL--3.0--only-536659"></a>
<a href="actions/workflows/build.yml"><img alt="Build" src="https://github.com/kenzok8/openwrt-usque/actions/workflows/build.yml/badge.svg"></a>
</p>

[Usque](https://github.com/Diniboy1123/usque) connects to Cloudflare WARP over MASQUE. This project provides OpenWrt packages, a native LuCI console, account management and TUN routing management. The upstream core stays at `v4.2.1`; no core code is modified.

Stable releases are published: [v2026.11.09](https://github.com/kenzok8/openwrt-usque/releases/tag/v2026.11.09) and [v2026.11.09-r2](https://github.com/kenzok8/openwrt-usque/releases/tag/v2026.11.09-r2). See the [documentation site](https://kenzok8.github.io/openwrt-usque/) for the user guide.

## What is this

A plugin that connects your router to Cloudflare WARP. The path is: device → Usque on the router → encrypted tunnel → Cloudflare edge → target website. To the website, your source becomes Cloudflare's egress IP.

No provider, no subscription, no Cloudflare account, and no hand-written configuration files needed.

| It can | It cannot |
| --- | --- |
| Change your egress IP; ordinary web browsing goes over the Cloudflare line | No subscriptions or node lists; you cannot pick a country |
| Encrypt this leg of the connection | No split routing; it cannot decide per-site whether to proxy or go direct |
| Local SOCKS5, or route a chosen LAN device's IPv4 through the tunnel | No geo-unblocking; streaming services still detect your region |
| Work with a few clicks in the UI | Not a full-device, full-protocol VPN: IPv6 is untouched, and ordinary local requests are not captured automatically |

If you need per-site routing, node selection, or provider subscriptions, use something like Clash / mihomo. Usque is just one fixed egress pipe: good enough and hassle-free, but no replacement for a full proxy setup.

### Three steps to get going

1. In Services → Usque, click **Register account** and accept the terms. Credentials are requested automatically; no Cloudflare sign-up is needed.
2. Pick a mode: SOCKS5 uses the default `127.0.0.1:1080`; to route a whole device, choose TUN and specify the LAN device (pre-filled by default).
3. Tick **Enable and autostart** → Save & Apply → click **Check connection**. It is only really up when you see `warp=on`.

<details>
<summary><b>About registration</b></summary>

When you click register, the plugin calls Cloudflare's free device-registration API and gets back a set of device credentials (device ID, private key, token, endpoint), stored in `/etc/usque/config.json`. It is not a device under your Cloudflare account, so it does not show up in the Cloudflare dashboard and cannot be deleted there; to change identity, just register again and the new credentials overwrite the old ones. This tier is free WARP, not WARP+.

Repeated registrations from the same broadband connection in a short time get rate-limited and fail. On failure the plugin keeps the old account; wait a while and try again instead of clicking repeatedly.

</details>

### Two common pitfalls

- **A passing check only means that one request got through**, not that the line is stable long-term. Speed on this route fluctuates; re-test first when it feels slow instead of changing the configuration right away.
- **Websites won't open after TUN takes over**: tick `Route LAN DNS into the tunnel` in settings. A common case is traffic entering the tunnel while domain names resolve to a bogus address, leaving pages spinning forever.

## Endpoints

By default it uses the Cloudflare WARP entry recorded in the account. When that entry IP is blocked by the line (typically all checks red while other websites work fine), pick another preset entry from the endpoint dropdown on the settings page; the account file is not touched.

Preset entries: `162.159.198.1`, `162.159.198.2`, `162.159.198.3`, `162.159.192.1`, `162.159.193.1`, `162.159.195.1`, `162.159.204.1`.

Field-tested: when `162.159.198.2` was blocked, `162.159.198.1` built the tunnel fine.

## Design points

- **Two ways to use it**: SOCKS5 for apps that use a proxy on purpose, TUN to take over a chosen LAN device's IPv4.
- **State shown clearly**: account, process and routing are displayed separately; the connection check runs three paths (WARP itself, an overseas site, a domestic site) with latency and signal bars — one success does not mean long-term stability.
- **Accounts are never deleted before replacement**: registration runs serially; the old file is only replaced after the new account verifies, and a failure keeps the old account.
- **Network resources have owners**: it refuses to occupy another program's policy table, mark or firewall configuration, and cleans up from the startup snapshot when stopped.
- **Lightweight UI**: native LuCI, no Node frontend runtime dependency, no online fonts, works on phones and with the host theme.

## Screenshots

<details open>
<summary><b>Desktop screenshots</b></summary>
<br>
<table>
<tr>
<td align="center"><b>Overview</b><br><img width="420" src="https://raw.githubusercontent.com/kenzok8/kenzok8/main/screenshot/usque/usque-overview.png"></td>
<td align="center"><b>Settings</b><br><img width="420" src="https://raw.githubusercontent.com/kenzok8/kenzok8/main/screenshot/usque/usque-settings.png"></td>
</tr>
<tr>
<td align="center"><b>Logs</b><br><img width="420" src="https://raw.githubusercontent.com/kenzok8/kenzok8/main/screenshot/usque/usque-logs.png"></td>
<td align="center"><b>Updates</b><br><img width="420" src="https://raw.githubusercontent.com/kenzok8/kenzok8/main/screenshot/usque/usque-updates.png"></td>
</tr>
</table>
</details>

## Installation and building

| Firmware | Package format | Build matrix |
| --- | --- | --- |
| OpenWrt / ImmortalWrt 24.10 | IPK | x86_64, four AArch64 variants, two ARMv7 variants |
| OpenWrt / ImmortalWrt 25.12 | APK | Same as above |

AArch64 package architectures are `aarch64_generic`, `aarch64_cortex-a53`, `aarch64_cortex-a72`, `aarch64_cortex-a76`; ARMv7 are `arm_cortex-a7_neon-vfpv4` and `arm_cortex-a9`. ARM picks the upstream armv5/v6/v7 binary by CPU. The MIPS mapping stays in the Makefile but is not in the build matrix, and its float ABI and on-device behavior are not yet verified.

Download the `usque` and `luci-app-usque` packages matching your firmware and architecture from [Releases](https://github.com/kenzok8/openwrt-usque/releases) (`luci-app-usque` is architecture-independent), plus the matching checksum file, and verify before installing:

```sh
sha256sum -c SHA256SUMS-24.10  # OpenWrt 24.10
sha256sum -c SHA256SUMS-25.12  # OpenWrt 25.12
```

Then install through the LuCI package page or the firmware package manager. Do not mix IPK, APK, or packages for another architecture. Dependencies should come from a firmware feed matching the current kernel — see below.

Alternatively, add this repository as a feed in the OpenWrt SDK and build `usque` and `luci-app-usque`; or use the repo's `Build OpenWrt packages` workflow, which only keeps build artifacts with a `SHA256SUMS` file — it does not publish or sign a package feed.

## Dependencies

| Package | Purpose |
| :--- | :--- |
| `kmod-tun` | TUN device |
| `ip-full` | Full iproute2, for policy routing and rule management |
| `ca-bundle` | CA certificate bundle, for curl to reach Cloudflare over HTTPS |
| `jsonfilter` | Parses JSON from registration responses and ubus output |
| `firewall4` | nftables firewall rules |
| `curl` | HTTPS requests for registration and connection checks |
| `flock` | Serializes registration |
| `ubus` | Queries service status and network interfaces |
| `coreutils-timeout` | Registration timeout control |
| `luci-base` | LuCI runtime dependency (`luci-app-usque`) |

See `usque/Makefile` and `luci-app-usque/Makefile` for the complete dependencies.

## Usage

The account lives in `/etc/usque/config.json`, directory mode `700`, file mode `600`. Status and check endpoints never return the account private key, token, or full configuration; do not paste that file into a public issue. Whether uninstalling keeps the configuration depends on the package manager — back it up yourself beforehand.

<details>
<summary><b>TUN traffic scope</b></summary>

- Pick a TUN name not used by another network; the LAN device should belong to the firewall `lan` zone, `br-lan` by default.
- With LAN takeover on, only forwarded IPv4 from the chosen device is handled. Private addresses, multicast and the WARP endpoint keep their original paths.
- **IPv6 is unchanged**; router DNS is unchanged too unless the DNS option is ticked.
- With **Route LAN DNS into the tunnel** ticked, the chosen device's DNS goes through the in-tunnel resolvers (`1.1.1.1` / `1.0.0.1`); other clients and the router's own DNS are unaffected. This works by temporarily rewriting the query destination port, without touching the system's existing DNS hijack rules, and it reverts automatically on stop.
- Do not treat it as a full-protocol, full-device VPN.
- The router's main default route is untouched. If the standalone table `257`, mark `0x7573`, or priorities `25700/25701` are taken by another program, takeover is refused.
- While enabled, a disappearing TUN keeps the blocking route and the taken-over IPv4 waits for reconnect; **the original path is restored after Usque stops**.
- Stopping manually does not change the autostart setting. Only unchecking Enable and saving & applying turns autostart off.

</details>

## Verification and troubleshooting

```sh
# Local source check, does not contact Cloudflare
bash scripts/check.sh

# Router status, returns no account credentials
/usr/share/usque/status.sh

# Manual connectivity check, does not register an account
/usr/share/usque/diagnostics.sh

# System log, check for sensitive info before sharing
logread -e usque
```

If the page still looks old after an upgrade or after replacing UI files (e.g. the overview page still carries the settings form), force-refresh once: `Cmd + Shift + R` on Mac, `Ctrl + Shift + R` on Windows.

If the process shows as running but checks fail, first verify the actual proxy address, port, TUN routing, and external network. Changing the listen address may expose an unauthenticated SOCKS5 to other reachable clients — only use it on a trusted network and restrict the firewall.

## Maintenance and documentation

- `.github/workflows/build.yml`: packaging and artifact checks across two SDKs and seven architectures.
- `.github/workflows/release.yml`: on a `v*` tag, runs the full matrix and creates a draft Release; the maintainer fills in the notes before publishing.
- `.github/workflows/upstream.yml`: checks the upstream stable release and prepares a candidate for review; it never changes main or publishes on its own.
- `.github/dependabot.yml`: checks GitHub Actions dependencies weekly.
- `.github/workflows/docs.yml`: generates the user guide from package metadata and deploys it to [GitHub Pages](https://kenzok8.github.io/openwrt-usque/).

Build the docs locally: `python3 scripts/build-docs.py`, output in `artifacts/docs/`. The docs site is a lightweight static guide with search, light/dark mode, mobile navigation and a 404 page.

## Credits and license

The core and prebuilt archives come from [Diniboy1123/usque v4.2.1](https://github.com/Diniboy1123/usque/releases/tag/v4.2.1), with SHA256 pinned from the upstream `checksums.txt`; the core is MIT-licensed. The OpenWrt integration scripts, LuCI interface and documentation are [GPL-3.0-only](LICENSE), and both licenses are installed with the packages.

This is an unofficial integration, with no affiliation with or endorsement by Cloudflare.
