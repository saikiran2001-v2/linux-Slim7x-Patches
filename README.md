# X1E80100 Linux Kernel Patches for Lenovo Yoga Slim 7x

This repository contains specific patch sets for the Lenovo Yoga Slim 7x (Snapdragon X Elite) running Linux.

## Branch Structure

Each feature or fix is isolated in its own branch to keep changes modular and easy to apply. Please switch to the relevant branch to access the patches for a specific feature.

### Available Branches

| Branch | Description | Status |
| :--- | :--- | :--- |
| **[camera](../camera)** | Patches for Camera functionality (OV02C10). | 🟢 **Stable** (Workaround)<br>Ideal Solution: Implement Hardware discharge in Rpmh regulator node. |
| **[deepsleep](../deepsleep)** | Patches for Deep Sleep (Suspend/Resume). | 🟢 **Stable**<br>Suspend drain is still little high compared to Windows.<br>Spurious wakes: Fixed. |
| **[sound](../sound)** | Patches for Audio support. | 🚧 **Partially Stable**<br>ADSP cannot be loaded (No support in upstream). |
| **[wifi](../wifi)** | Patches for WiFi 5GHz and instability. | 🟢 **Stable** (Workaround)<br>Ideal solution: Fix the issue in mm subsystem. |
| **[usb-c-display](../usb-c-display)** | Patches for USB-C DisplayPort (External Monitor) vblank value struck after long suspend. | 🚧 **Partially Stable** |

## Test Setup

These patches have been developed and tested on the following configuration:

- **Device:** Lenovo Yoga Slim 7x (14Q8X9)
- **Variant:** 16GB RAM / 512GB SSD
- **Kernel Version:** 7.0-rc1

## Usage

To use these patches, clone the repository and checkout the desired branch:

```bash
git clone https://github.com/saikiran2001/linux-patches.git
cd linux-patches

# For Camera patches
git checkout camera

# For WiFi patches
git checkout wifi
```

## Disclaimer

**USE AT YOUR OWN RISK.**

The patches provided in this repository are experimental and intended for development and testing purposes only. I am **NOT** responsible for any damage to your hardware, data loss, or system instability that may result from applying these patches. This includes, but is not limited to:
- Speaker damage due to incorrect gain settings.
- Hardware failure due to power management changes.
- Data corruption due to filesystem crashes.

By using these patches, you agree that you are doing so entirely at your own discretion and risk.

---
**Note:** The `main` branch only contains this documentation.