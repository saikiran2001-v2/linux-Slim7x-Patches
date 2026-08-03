# Slim 7x audio fixes

This branch contains the ASoC changes used by the Lenovo Yoga Slim 7x
(X1E80100), including the latest LPASS WSA runtime-PM and v2.5 register fixes.

## Patch order

```text
0001-ASoC-qcom-x1e80100-Add-Dell-XPS13-9345-support.patch
0002-ASoC-qcom-x1e80100-Add-Lenovo-Yoga-Slim-7x-Support.patch
0003-ASoC-codecs-lpass-wsa-macro-Switch-to-PM-clock-frame.patch
0004-ASoC-codecs-lpass-wsa-macro-Guard-optional-NPL-clock.patch
0005-ASoC-codecs-lpass-wsa-macro-use-sparse-flat-regcache.patch
0006-ASoC-codecs-lpass-wsa-macro-fix-v2.5-speaker-mode-of.patch
```

Patch 1 introduces the machine-driver configuration structure used by patch 2.
The remaining patches fix WSA clock/runtime-PM handling, tolerate an optional
NPL clock, reduce the flat regcache allocation, and use the correct LPASS v2.5
register offsets for speaker mode.

The old `qcom,broken-reset` revert is no longer part of this branch. The former
machine-driver “topology loading” patch is also intentionally absent: it only
requested and released the firmware file and did not load it into the DSP. The
working AudioReach topology path remains available through the normal audio
stack.

## Apply

```bash
git am /path/to/s7x-patches/*.patch
```

## Firmware and UCM prerequisites

Build the Slim 7x topology from
[linux-msm/audioreach-topology](https://github.com/linux-msm/audioreach-topology)
and install it using the exact firmware name expected by the current topology
and UCM configuration. Install a recent
[alsa-ucm-conf](https://github.com/alsa-project/alsa-ucm-conf) containing the
X1E80100 Lenovo profile, then restart the user audio stack:

```bash
systemctl --user restart pipewire pipewire-pulse wireplumber
```

Verify card and topology initialization with:

```bash
cat /proc/asound/cards
aplay -l
journalctl -k -b | grep -Ei 'audio|audioreach|tplg|wsa'
```

## Safety and status

Tested on a Lenovo Yoga Slim 7x 14Q8X9 with Linux 7.2-rc6 / Silvercore 1.3.
Audio registers and works, and the former WSA register `0x5dc` errors are gone.
Speaker volume remains lower than Windows and calibration/protection is not
fully equivalent to the vendor stack. Keep conservative UCM gain values and do
not remove speaker-safety limits merely because a topology file is installed.
