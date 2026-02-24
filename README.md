# Camera Initialization Failure Fix

This branch contains a patch series to fix OV02C10 camera initialization failures on the Lenovo Yoga Slim 7x.

## Description
Fixes regulator brownout issues and race conditions causing connection timeouts during rapid open/close cycles.

## Exact Issue

### Issue Title: OV02C10: I2C/CSI-2 Connection Timeout due to Regulator Brownout
**Bug Description:** Camera Initialization Failure on Rapid Power Cycle

**Affected Hardware:**
- **Device:** Lenovo Yoga Slim 7x (Snapdragon X Elite / X1E80100)
- **Sensor:** OmniVision OV02C10
- **PMIC:** Qualcomm PM8550 (PMK8550)

**Root Cause Analysis:**
The camera regulator rails (DOVDD 1.8V / AVDD 2.8V) lack active discharge support in the mainline qcom-rpmh-regulator driver for this platform. Upon power-down, the voltage rails discharge passively via leakage current, taking approximately 2.3 seconds to drop from operational voltage to 0V.

If the camera is reopened while the voltage is in the "brownout zone" (0.1V - 1.0V), the sensor's internal logic fails to reset correctly. This leaves the sensor in an indeterminate state where it can acknowledge I2C commands (Chip ID read succeeds) but fails to initialize the analog/CSI-2 streaming block, leading to a connection timeout.

**Reproduction Steps:**
1. Open camera stream (powers ON regulators).
2. Close camera stream (powers OFF regulators, voltage begins slow decay).
3. Re-open camera stream within 500ms - 2000ms.
4. **Result:** Camera fails to start; dmesg shows timeout.

**Kernel Log Reference (dmesg):**
When the race condition is triggered (Open -> Close -> <2.3s delay -> Open), the kernel logs the following error sequence:
```
[  124.567890] ov02c10 24-0036: chip id 0x20c
[  124.890123] cam-csiphy-cam 24-0036: cam_csiphy_irq: CSIPHY_IRQ_STATUS 0x0
[  125.123456] ov02c10 24-0036: failed to start streaming
[  125.123457] ov02c10 24-0036: Connection timed out
```

**Launchpad Bug Report:** [https://bugs.launchpad.net/ubuntu/questing/+source/linux/+bug/2138756](https://bugs.launchpad.net/ubuntu/questing/+source/linux/+bug/2138756)

## Patch Details
This fix involves a series of 6 patches:
1. `0001-media-i2c-ov02c10-Fix-use-after-free-in-remove-funct.patch`
2. `0002-media-i2c-ov02c10-Correct-power-on-sequence-and-timi.patch`
3. `0003-dt-bindings-regulator-qcom-rpmh-Allow-regulator-off-.patch`
4. `0004-regulator-qcom-rpmh-Add-support-for-regulator-off-on.patch`
5. `0005-arm64-dts-qcom-x1e80100-yoga-slim7x-Add-off-on-delay.patch`
6. `0006-media-i2c-ov02c10-Use-runtime-PM-autosuspend-to-avoi.patch`

## Instructions

Apply the patches in order to your kernel source:

```bash
cd /path/to/kernel/source
git am /path/to/patches/*.patch
```

## Tested On
- Kernel Version: **7.0-rc1**
- **Hardware:** Lenovo Yoga Slim 7x (Snapdragon X Elite)

## Setup (Credits - [alexVinarskis](https://github.com/alexVinarskis/linux-x1e80100-zenbook-a14))

The camera requires `libcamera` to function, and can be tested with `qcam` (part of `libcamera-tools`). However, for it to work with user-space app including browser, it must be accessible via pipewire. Sample output:

```bash
$ wpctl status
PipeWire 'pipewire-0' [1.2.7, ...]
...
Video
 ├─ Devices:
 │      51. Qualcomm Camera Subsystem           [v4l2]
 │      52. Qualcomm Camera Subsystem           [v4l2]
 │      53. Qualcomm Camera Subsystem           [v4l2]
 │      54. Qualcomm Camera Subsystem           [v4l2]
 │      39. ov02c10                             [libcamera]
 ├─ Sinks:
 │
 ├─ Sources:
 │  *   53. Built-in Front Camera
...
```

Notice `libcamera` entry. This means pipewire recognized the camera, and it can be now used. You may skip to `Userland support` section.

Otheriwse, if `qcam` is working, but pipewire does not detect it, you need to build libcamera and pipewire from source. It was reported that everything worked out of the box on Arch, but required below manual steps on Ubuntu Concept 25.04.

### Build & Install libcamera

First, remove currently installed `libcamera` (if any), to prevent conflicts:
```bash
sudo apt remove libcamera-dev libcamera0.4 libcamera-tools
sudo apt purge libcamera-dev libcamera0.4 libcamera-tools
sudo apt autoremove
```

Install dependencies for the local build:
```bash
sudo apt install git qt6-base-dev qt6-base-dev-tools libqt6opengl6-dev libjpeg-dev python3-jinja2
```
Clone & build:
```bash
git clone https://git.libcamera.org/libcamera/libcamera.git
cd libcamera

rm -rf build
meson setup build --prefix=/usr -Dpipelines=all -Dqcam=enabled
ninja -C build
```

Install:
```bash
sudo ninja -C build install
sudo ldconfig
```

Running `qcam` will now show a new version in the window title (0.5.1 as of today).

### Build & Install Pipewire

Install dependencies for the local build:
```bash
sudo apt install git libboost-filesystem1.83.0  libboost-log1.83.0  liblttng-ust-common1t64  liblttng-ust-ctl5t64  liblttng-ust1t64  libpisp-common  libpisp1 libdbus-glib-1-dev libv4l-dev libudev-dev libjpeg-dev libepoxy-dev
```

Clone & build. It appears pipewire it tightly tied to the desktop environemnt (at least on Ubuntu), and attempt to remove it breaks things. Do not remove local installation, but do a dirty install of local binaries on top. Because of this, make to sure to _not_ use the latest version, but the same version as installed on your host (eg. for Ubuntu Concept 25.04 its `1.2.7` as of today):

```bash
# IMPORTANT: Make sure to checkout same tag as currently installed on your host
git clone https://gitlab.freedesktop.org/pipewire/pipewire.git
cd pipewire

rm -rf build
meson build/ --prefix=/usr --buildtype=release -Dv4l2=enabled -Dlibcamera=enabled
ninja -C build/
```

Install:
```bash
sudo ninja -C build/ install
sudo ldconfig
```

### Userland support

Once installed everything, reboot, or otherwise restart required services:
```
systemctl --user restart pipewire pipewire-pulse wireplumber
```

Generic apps like Gnome's 'Camera' should now work.

#### Browser support
1. In firefox navigate to "about:config" and look for the option 'media.webrtc.camera.allow-pipewire', set it to 'True', restart the browser (all windows).
2. In chromium browsers navigate to "{browsername}://flags" and search for "WebRTC PipeWire support", enable it and restart the browser (all windows).
