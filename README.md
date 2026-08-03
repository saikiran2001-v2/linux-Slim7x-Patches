# USB-C DisplayPort hotplug and resume fixes

This branch contains the MSM DisplayPort, DPU and Parade PS883x changes carried
for USB-C displays on the Lenovo Yoga Slim 7x.

## Problems addressed

- disconnect/reconnect crashes after a long suspend;
- a wedged DPU encoder hanging the CPU during atomic disable;
- unsafe DP cleanup after wedge detection;
- inactive DisplayPort audio endpoints failing userspace probing;
- cached EDIDs producing no modes;
- DPU performance state not being restored on runtime resume; and
- unreliable DisplayPort fallback through PS883x USB4-capable docks.

The PS883x USB4 disable is a platform workaround until the X1E USB4 controller
and combo-PHY stack support DP tunnelling correctly. USB3 plus classic DP
Alt Mode remains available.

## Patch order

There are 11 numbered patches. Apply all of them in filename order:

```bash
git am /path/to/s7x-patches/*.patch
```

The series starts with the Slim 7x hotplug/wedge recovery changes, follows with
DP audio, EDID and runtime-resume fixes, and finishes with the PS883x retimer
and device-tree workaround. The unrelated media change from the historical
`fix build` development commit is intentionally not exported.

## Reproduction background

A typical failure sequence was:

1. connect an external display over USB-C;
2. suspend for an extended period;
3. resume with the display pipeline no longer responding;
4. disconnect or reconnect the display; and
5. hit vblank/flush timeouts followed by a watchdog reboot.

The wedge patches avoid further unsafe MMIO accesses once the encoder is known
to be stuck and perform software/controller cleanup through a safer path.

## Validation status

Built and boot-tested as part of Linux 7.2-rc6 / Silvercore 1.3 on a Lenovo
Yoga Slim 7x 14Q8X9. Internal display, suspend/resume and normal desktop use are
stable. USB-C DisplayPort remains an experimental area: test the exact monitor,
dock, cable, hotplug order and long-suspend scenario you rely on.

Useful diagnostics:

```bash
journalctl -k -b | grep -Ei 'drm|dpu|dp |ps883|typec|vblank|wedg|timeout'
cat /sys/class/drm/*/status
```

Keep a known-good kernel installed because the wedge recovery code is a
defensive workaround around a hardware/driver pipeline failure.
