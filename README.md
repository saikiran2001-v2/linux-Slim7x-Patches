# Deep Sleep (Suspend/Resume) Fixes

This branch contains patches to improve deep sleep (s2idle) reliability and prevent spurious wakeups on the Lenovo Yoga Slim 7x.

## Description
Fixes issues where the device would wake up immediately after suspending, particularly when a charger is connected or due to improper power management of HID devices (Touchpad/Keyboard/Touchscreen).

## Exact Issues

### 1. Charger Wakeup Fix
**Issue:**
When a charger is connected, the PMIC sends frequent updates via GLINK for battery status, UCSI notifications, and Type-C events. These messages arrive as interrupts on the `apps_rsc`, causing the system to wake up immediately from s2idle or preventing entry entirely.

**Fix:**
- `0001-soc-qcom-pmic_glink-suspend-charger-updates-to-preve.patch`
- Adds suspend/resume callbacks to the `pmic_glink` driver.
- Sets a 'suspended' flag during suspend which causes the driver to unconditionally drop all incoming messages, effectively masking these wake sources while the system is asleep.

### 2. I2C HID Regulator Supply Fix
**Issue:**
Input devices (Touchpad, Keyboard, Touchscreen) were falling back to dummy regulators because `vdd` and `vddl` supplies were not defined in the Device Tree. This prevented proper power management and caused IRQ affinity failures (`-EINVAL`) during CPU offline processes in suspend, leading to spurious wakeups.

**Fix:**
- `0002-dt-x1e-add-vdd-vddl-supplies-to-I2C-HID-nodes.patch`
- Adds `vdd-supply = <&vreg_l8b_3p0>` and `vddl-supply = <&vreg_l15b_1p8>` to the I2C HID nodes in `x1e80100-lenovo-yoga-slim7x.dts`.

## Patch Details
- `0001-soc-qcom-pmic_glink-suspend-charger-updates-to-preve.patch`
- `0002-dt-x1e-add-vdd-vddl-supplies-to-I2C-HID-nodes.patch`

## Instructions

Apply the patches to your kernel source:

```bash
cd /path/to/kernel/source
git am /path/to/patches/*.patch
```

## Tested On
- Kernel Version: **7.0-rc1**
- **Hardware:** Lenovo Yoga Slim 7x (Snapdragon X Elite)