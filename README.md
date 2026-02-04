# USB-C DisplayPort Hotplug Fix

This branch contains a patch to fix USB-C DisplayPort hotplug functionality on the Lenovo Yoga Slim 7x.

## Description
Fixes crashes or failures when hotplugging an external monitor via USB-C after long suspend.

## Exact issue
1. Suspend the device for more than 10 minutes while an external monitor is connected.
2. Wake the device up.
3. External monitor will not be detected.
4. Disconnect the external monitor (Inorder to reconnect).
5. The device will reboot.

## Patch Details
- **File**: `0001-drm-msm-dp-Fix-USB-C-DisplayPort-hotplug-crash-after.patch`

## Instructions

Apply the patch directly to your kernel source:

```bash
cd /path/to/kernel/source
git am < /path/to/patches/0001-drm-msm-dp-Fix-USB-C-DisplayPort-hotplug-crash-after.patch
```

## Tested On
- Kernel Version: **6.19-rc8**