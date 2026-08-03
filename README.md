# ath12k suspend/resume fixes

This branch contains the ath12k PCI/MHI fixes used on the Lenovo Yoga Slim 7x
with its WCN7850 adapter.

## Patch order

```text
0001-wifi-ath12k-Use-pci_-enable-disable-_link_state-APIs.patch
0002-wifi-ath12k-fix-CMA-error-and-MHI-state-mismatch-dur.patch
```

The first patch moves ath12k ASPM changes to the PCI core APIs so that the PCI
core and hardware remain synchronized. The second preserves MHI DMA tables and
state across suspend, avoiding CMA page corruption and the resume-time
`failed to set mhi state INIT` failure.

## Symptoms fixed

- bad-page/CMA corruption during resume;
- Wi-Fi failing to return after suspend;
- MHI INIT state mismatch during resume; and
- direct ASPM register manipulation bypassing PCI core state.

The old experimental PME-based “spurious wakeup” patch is not included. It was
reverted and is not present in the tested kernel.

## Apply

```bash
git am /path/to/s7x-patches/*.patch
```

## Validation

Tested on a Lenovo Yoga Slim 7x 14Q8X9, WCN7850 hw2.0, with Linux 7.2-rc6 /
Silvercore 1.3. Wi-Fi reconnects after multi-minute s2idle cycles. A large QMI
DMA allocation may still fail and fall back to a smaller allocation; that
fallback is non-fatal and is distinct from the fixed CMA corruption.

Useful post-resume checks:

```bash
journalctl -k -b | grep -Ei 'ath12k|mhi|CMA|Bad page state'
ip link show
```
