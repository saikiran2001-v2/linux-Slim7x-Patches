# OV02C10 camera power-sequencing fixes

This branch contains the currently tested OV02C10 stability fixes for the
Lenovo Yoga Slim 7x (X1E80100). The series targets the intermittent camera
failure seen after closing and reopening the camera.

## Root cause and fix

The earlier version of this branch assumed that the camera rails needed a
multi-second discharge delay and added an `off-on-delay-us` regulator
workaround. Testing showed that this was not the correct fix. The working
solution is to keep the sensor, CCI controller and pinctrl states in a valid
firmware-defined power sequence:

1. fix the OV02C10 remove-path use-after-free;
2. correct regulator, clock and reset ordering;
3. use runtime-PM autosuspend to avoid unnecessary rapid power cycling;
4. select the CCI sleep/default pinctrl states during runtime PM, preventing
   the powered-off sensor from being back-fed through SDA/SCL; and
5. honour firmware power sequencing, including the reset GPIO polarity.

The old RPMh regulator binding/driver changes and the 2.3/3-second device-tree
delay have been removed from this series.

## Patch order

```text
0001-media-i2c-ov02c10-Fix-use-after-free-in-remove-funct.patch
0002-media-i2c-ov02c10-Correct-power-on-sequence-and-timi.patch
0003-media-i2c-ov02c10-Use-runtime-PM-autosuspend-to-avoi.patch
0004-i2c-qcom-cci-select-pinctrl-states-during-runtime-PM.patch
0005-media-ov02c10-follow-firmware-power-sequencing.patch
```

Apply the complete series in order:

```bash
git am /path/to/s7x-patches/*.patch
```

## Prerequisites

The target kernel tree must already contain the Slim 7x camera device-tree
enablement (PM8010 camera PMIC, OV02C10/OV02E10 sensor node, correct RGB
camera supplies and privacy indicator). These patches are the stability fix,
not the initial board enablement.

## Validation

Tested on a Lenovo Yoga Slim 7x 14Q8X9 with Linux 7.2-rc6 / Silvercore 1.3:

- camera smoke test passed;
- 20 rapid Kamoso open/close cycles passed;
- 20 forced power cycles with a four-second interval passed;
- no CCI queue timeouts or mode failures were logged; and
- the sensor and CCI controller runtime-suspended after every cycle.

Bug background: [Launchpad #2138756](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/2138756).

## Userspace

The camera also needs a libcamera pipeline and PipeWire/WirePlumber camera
integration. Confirm that `wpctl status` lists the OV02C10 libcamera device
and a camera source. Firefox-based browsers may additionally require
`media.webrtc.camera.allow-pipewire=true`.
