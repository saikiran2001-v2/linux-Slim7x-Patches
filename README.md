# AudioReach Topology Fix

This branch contains patches to enable AudioReach topology loading on the Lenovo Yoga Slim 7x.

## Description
Enables the loading of AudioReach topology files required for sound functionality on the Qualcomm Snapdragon X Elite platform.

## Exact Issue

### Bug Title: AudioReach: Topology loading fails with firmware path resolution error on Snapdragon X Elite platforms

**Log Reference:**
```
[    8.350070] snd-x1e80100 sound: Loading topology from qcom/x1e80100/LENOVO/83ED/LenovoSlim7x-tplg.bin
[    8.350XXX] snd-x1e80100 sound: error: failed to load topology: -2
```

## Patch Details
- `0001-Revert-arm64-dts-qcom-x1-el2-Add-qcom-broken-reset-f.patch`
- `0002-ASoC-qcom-x1e80100-Add-Lenovo-Yoga-Slim-7x-Support.patch`
- `0003-ASoC-qcom-x1e80100-Add-topology-file-loading-support.patch`

## Instructions

Apply the patches to your kernel source:

```bash
cd /path/to/kernel/source
git am /path/to/patches/*.patch
```

## Setup Instructions

**Important:** This patch series requires specific topology and UCM configuration files to function.

### 1. Audioreach-topology
Latest `linux-next` contains required binaries. Alternatively, compile from source as follows:

* Download latest sources with Lenovo Yoga Slim 7x support from [https://github.com/linux-msm/audioreach-topology/](https://github.com/linux-msm/audioreach-topology/)
* Build via:
```bash
cmake .
cmake --build .
```

### 2. AudioReach Topology Installation

**Files to install:**

1. **Topology binary** (`.bin`):
   
   Copy and rename the binary to `LenovoSlim7x-tplg.bin`:
   ```bash
   sudo cp X1E80100-LENOVO-Yoga-Slim7x-tplg.bin \
     /lib/firmware/updates/qcom/x1e80100/LENOVO/83ED/LenovoSlim7x-tplg.bin
   
   sudo chmod 644 /lib/firmware/updates/qcom/x1e80100/LENOVO/83ED/LenovoSlim7x-tplg.bin
   ```

2. **UCM configuration** (`.conf`):
   ```bash
   sudo mkdir -p /usr/share/alsa/ucm2/conf.d/x1e80100/LENOVO/83ED/
   
   sudo cp X1E80100-LENOVO-Yoga-Slim7x.conf \
     /usr/share/alsa/ucm2/conf.d/x1e80100/LENOVO/83ED

   sudo chmod 644 /usr/share/alsa/ucm2/conf.d/x1e80100/LENOVO/83ED/X1E80100-LENOVO-Yoga-Slim7x.conf
   ```

3. ALSA Configuration

* Download the latest configuration with Lenovo Yoga Slim 7x support from [https://github.com/alsa-project/alsa-ucm-conf](https://github.com/alsa-project/alsa-ucm-conf)
* Follow the instructions in the repository's `README.md` to install.

**Important:** To avoid potential hardware damage, lower the gain settings before use. Change the value `84` to `5` in the following files:
- `/usr/share/alsa/ucm2/codecs/qcom-lpass/wsa-macro/four-speakers/init.conf`
- `/usr/share/alsa/ucm2/codecs

**Note**: If you are on kernel with latest tag in sound, the sound will be low and then you can experiment with this value gradually without damaging your hardware. 

4. **Reload audio stack:**
   ```bash
   systemctl --user restart pipewire pipewire-pulse wireplumber
   ```

5. **Verify:**
   ```bash
   # Check topology loaded
   dmesg | grep -i tplg
   
   # Check card detected
   aplay -l
   
   # Check UCM
   alsaucm listcards
   ```

## Tested On
- Kernel Version: **7.0-rc1**
- **Hardware:** Lenovo Yoga Slim 7x (Snapdragon X Elite)