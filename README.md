# USB-C DisplayPort Hotplug Fix

This branch contains a patch to fix USB-C DisplayPort Wedge kernel panic on the Lenovo Yoga Slim 7x.

## Description
Fixes kernel crash when hotplugging an external monitor via USB-C after long suspend.

## Exact issue
1. Suspend the device for more than 10 minutes while an external monitor is connected.
2. Wake the device up.
3. External monitor will not be detected.
4. Disconnect the external monitor (Inorder to reconnect).
5. The device will reboot.

## Patch Details
- **File**: `0001-drm-msm-dpu-Implement-wedge-detection-and-safe-recov.patch`

## Instructions

Apply the patch directly to your kernel source:

```bash
cd /path/to/kernel/source
git am < /path/to/patches/0001-drm-msm-dpu-Implement-wedge-detection-and-safe-recov.patch
```

## Tested On
- Kernel Version: **7.0-rc1**