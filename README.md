# Deep-sleep and suspend/resume fixes

This branch contains the suspend-related fixes currently carried for the
Lenovo Yoga Slim 7x (X1E80100).

## Patch order

```text
0001-soc-qcom-pmic_glink-suspend-charger-updates-to-preve.patch
0002-dt-x1e-add-vdd-vddl-supplies-to-I2C-HID-nodes.patch
0003-HID-lenovo-Add-suspend-resume-to-turn-off-LEDs-durin.patch
0004-UBUNTU-SAUCE-platform-arm64-Add-driver-for-Lenovo-Yo.patch
0005-platform-arm64-lenovo-yoga-slim7x-Add-keyboard-backl.patch
```

The series:

- suppresses PMIC GLINK charger/type-C updates while the system is suspended;
- supplies the Slim 7x I2C HID devices from their real regulators instead of
  dummy regulators;
- turns off Lenovo HID LEDs during suspend;
- adds the Slim 7x embedded-controller platform driver; and
- saves, disables and restores the keyboard backlight across suspend.

The old `net: ath12k: fix spurious wakeups during suspend` patch has been
removed. It was reverted during development and is not present in the tested
kernel. Wi-Fi suspend/resume fixes are maintained on the `wifi` branch.

## Apply

```bash
git am /path/to/s7x-patches/*.patch
```

The EC/backlight patches expect the corresponding Slim 7x EC device-tree node
to exist in the target X1E tree.

## Validation

Tested on a Lenovo Yoga Slim 7x 14Q8X9 with Linux 7.2-rc6 / Silvercore 1.3.
Repeated s2idle and multi-minute suspend cycles complete successfully. The
kernel was also tested after removing `pd_ignore_unused`; power domains still
synchronized correctly on this 7.2-rc kernel.

Useful checks after applying the series:

```bash
cat /sys/power/mem_sleep
journalctl -b | grep -E 'PM: suspend|PM: resume'
cat /sys/kernel/debug/wakeup_sources
```

`[s2idle]` should be selected in `mem_sleep`. Review wakeup sources before
assuming that a short suspend was caused by a kernel failure.
