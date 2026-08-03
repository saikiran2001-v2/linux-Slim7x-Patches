# Linux patches for the Lenovo Yoga Slim 7x

This repository publishes the out-of-tree Linux patches currently used on the
Lenovo Yoga Slim 7x 14Q8X9 (Snapdragon X Elite / X1E80100). Each feature is
kept on its own branch so the series can be reviewed and applied independently.

## Branches

| Branch | Scope | Current status |
| :--- | :--- | :--- |
| [`camera`](../camera) | OV02C10 lifetime, runtime-PM, CCI pinctrl and firmware power sequencing | Tested: rapid open/close and forced power-cycle loops pass |
| [`deepsleep`](../deepsleep) | PMIC GLINK, I2C HID supplies, Lenovo HID LEDs and EC keyboard backlight suspend handling | Stable in repeated multi-minute s2idle tests |
| [`sound`](../sound) | Slim 7x machine support and LPASS WSA runtime-PM/regcache/v2.5 fixes | Working with conservative speaker safety; volume remains low |
| [`wifi`](../wifi) | ath12k ASPM and MHI/CMA suspend-resume fixes | Stable across suspend/resume; large QMI DMA fallback remains non-fatal |
| [`usb-c-display`](../usb-c-display) | MSM DP hotplug/wedge recovery, resume and PS883x dock fixes | Experimental; exact dock/monitor combinations still need testing |

Each branch has its own README describing patch order, prerequisites, testing
and known limitations.

## Tested baseline

- **Device:** Lenovo Yoga Slim 7x 14Q8X9
- **Variant:** 16 GB RAM / 512 GB SSD
- **Kernel:** Linux 7.2-rc6, Silvercore 1.3
- **Architecture:** arm64

The patch files were regenerated from the commits carried by the tested
`7.2-rc6-x1e` kernel and apply cleanly to the Linux 7.2-rc6 baseline. Board
enablement prerequisites from an X1E development tree may still be required;
see the README on each branch.

## Usage

Clone the repository and check out only the feature branch you need:

```bash
git clone https://github.com/saikiranworks/linux-Slim7x-Patches.git
cd linux-Slim7x-Patches
git switch camera
git am ./*.patch
```

Do not combine branches blindly. Inspect overlapping device-tree, ath12k,
audio and display changes against your target kernel first. Keep a known-good
distribution kernel available in the boot menu.

## Important changes from the old 7.0-rc1 export

- the camera regulator `off-on-delay` workaround was removed and replaced by
  CCI pinctrl plus correct firmware power sequencing;
- the experimental ath12k PME/spurious-wakeup patch was removed;
- the no-op custom AudioReach topology requester was removed while normal
  topology support remains; and
- the USB-C display branch now contains the complete carried hotplug, wedge,
  resume, EDID, audio and PS883x series.

## Disclaimer

These patches are development work and are provided without warranty. Kernel,
power-management, camera, audio and display changes can cause crashes, data
loss or hardware damage. In particular, do not raise speaker gain without
proper calibration and protection. Use the patches at your own risk.
