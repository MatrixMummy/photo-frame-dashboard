# Rayswitch RSFM101 10.1" (Frameo, RK3126, Android 6.0.1)

Record of one unit, set up 2026-10-15 from a Windows host. Observations are kept separate from
guesses. Unique identifiers (serial, MAC, IP, ADB key, backup hash) are kept out of this file.

Same SoC and Android version as the [BIUFRAME](biuframe.md), [Veidoo F10](veidoo-f10.md) and
[Rollei Smart Frame 107](rollei-smartframe-107.md), but a different vendor build and partition
layout. It ships with WebView 106 in `/system`, so the dashboard runs without a WebView update.

## Observations (read-only unless noted)

### Identity

| Parameter | Value |
| --- | --- |
| Retail name | Rayswitch RSFM101 10.1" (Frameo) |
| Brand / Model | `Rayswitch` / `Rayswitch_RSFM101` (`ro.product.device` and `ro.product.name` the same) |
| Firmware ID | `FrameO.DP104.v20260312`, built 2026-03-12 |
| Build Fingerprint | `rockchip/rk312x/rk312x:6.0.1/MXC89K/user.zcx.20260312.081202:user/release-keys` |
| SoC / Platform | Rockchip RK3126 (device tree `rockchip,rk3126`; `ro.board.platform=rk312x`, `ro.product.board=rk30sdk`, `ro.hardware=rk30board`), 32-bit ARMv7 |
| Kernel | Linux 3.10.0, built by `zcx@bnd106` |
| Android | 6.0.1, API 23, security patch 2019-11-05, `user` build, `ro.debuggable=0` |
| SELinux | Permissive |
| Display | 1280x800, 160 dpi, landscape |
| RAM / Storage | 1 GB RAM (not a low-RAM build), eMMC `mmcblk0` of 62,160,896 sectors (31.8 GB), userdata 27.6 GB |
| WebView | `com.android.webview` 106.0.5249.126, shipped in `/system/app/webview` |
| Wi-Fi | Onboard AIC8800 (`aic8800_fdrv`, SDIO driver `aicwf_sdio`) |
| SystemUI | Installed and enabled |
| Shipped Home | Frameo: system copy v1.29.14, updated in `/data` to v1.32.15 on 2026-10-15 |
| USB | `2207:0006`, `sys.usb.config=mtp,adb` |

### USB, ADB and root

- **ADB:** USB, with the key check on (`ro.adb.secure=1`); this computer's key is in
  `/data/misc/adb/adb_keys`. How ADB was switched on (Frameo's Beta Program toggle or already on)
  was not recorded.
- **Network ADB:** nothing listens on port 5555 and `service.adb.tcp.port` is unset.
- **Root:** shipped `/system/xbin/su` (SuperSU 2.45). `adb shell su -c id` returns `uid=0(root)`
  with no prompt.

### Partition layout (512-byte sectors)

| Name | Start | Sectors |
| --- | --- | --- |
| uboot | 16384 | 8192 |
| misc | 24576 | 8192 |
| resource | 32768 | 32768 |
| kernel | 65536 | 24576 |
| boot | 90112 | 45056 |
| recovery | 135168 | 65536 |
| backup | 200704 | 131072 |
| cache | 331776 | 262144 |
| metadata | 593920 | 32768 |
| kpanic | 626688 | 8192 |
| system | 634880 | 2359296 |
| radical_update | 2994176 | 131072 |
| userdata (`mmcblk0p13`) | 3125248 | 59035648 |

[`scripts/biuframe-setup.sh`](../scripts/biuframe-setup.sh) has no profile for this fingerprint, so
`check` stops at "unrecognized build fingerprint". Don't force a profile: the BIUFRAME length
(3,891,200 sectors) would read 374 MiB into `userdata` here, and the Akimart one (2,859,008) stops
partway through `system`. Run the [BIUFRAME guide](biuframe-setup-guide.md) steps by hand with this
frame's numbers.

### Vendor packages

Present and enabled: `com.adups.fota` (Adups FOTA), `android.rockchip.update.service`,
`com.DeviceTest`. They were left alone on this unit. The [Veidoo F10 notes](veidoo-f10.md#vendor-packages)
have the disable and undo commands for the same packages.

## Changes on this unit

1. **Backup (verified):** eMMC sectors 0 to 3,125,247 (everything before `userdata`, 1,600,126,976
   bytes) as a gzip image. Byte count, SHA-256 and gzip readback checked; no all-`0xCC` sector; `boot`
   and `recovery` start with `ANDROID!`. Independent re-reads of loader, uboot, kernel, boot,
   recovery, radical_update and 16 MiB of system matched. Restore was not tested; it would need
   Rockchip loader mode and `rkdeveloptool`.
2. **Kiosk:** Home HQ 1.1 from [`dashboard/kiosk`](../dashboard/kiosk/README.md), installed with
   its Home alias enabled. Built with `d8` from build-tools 36.1.0 (build-tools 34's `d8` crashed on
   JDK 21) and signed with the v1 scheme, which Android 6 needs.
3. **Frameo disabled** (`pm disable-user`), so a reboot opens Home HQ. Frameo's data was kept.
   After this, the kiosk's **Open launcher** opens Android Settings, because no other Home app is
   left. USB ADB still connects with Frameo disabled. Whether Android Settings → Apps was reached
   by hand on the panel ([README rule 9](../README.md#rules-we-learned-the-hard-way)) was not
   recorded.

### Undo

```sh
adb shell su -c 'pm enable net.frameo.frame'
adb shell am start -n com.wesbos.homehq/.MainActivity --ez home false
adb uninstall com.wesbos.homehq
```
