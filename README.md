# X1E80100 Linux Kernel Patches for Lenovo Yoga Slim 7x

This repository contains specific patch sets for the Lenovo Yoga Slim 7x (Snapdragon X Elite) running Linux.

## Branch Structure

Each feature or fix is isolated in its own branch to keep changes modular and easy to apply. Please switch to the relevant branch to access the patches for a specific feature.

### Available Branches

| Branch | Description | Status |
| :--- | :--- | :--- |
| **[camera](../tree/camera)** | Patches for Camera functionality (OV02C10). | 🟢 Stable |
| **[deepsleep](../tree/deepsleep)** | Patches for Deep Sleep (Suspend/Resume). | 🚧 In Progress |
| **[sound](../tree/sound)** | Patches for Audio support. | ⚠️ **Known Issue:** ADSP cannot be loaded (no kernel support yet). |
| **[wifi](../tree/wifi)** | Patches for WiFi 5GHz and instability. | 🟢 Stable |
| **[usb-c-display](../tree/usb-c-display)** | Patches for USB-C DisplayPort (External Monitor) Hotplug. | 🟢 Stable |
| **[misc](../tree/misc)** | Miscellaneous fixes and improvements. | 🟢 Stable |

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

## Current Known Issues

- **Sound**: Audio subsystem requires ADSP firmware/driver support which is currently missing in the mainline kernel for this specific SoC variant. 

---
**Note:** The `main` branch only contains this documentation.