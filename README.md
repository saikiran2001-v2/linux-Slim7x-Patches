# WiFi Suspend/Resume Fix

This branch contains a critical fix for WiFi (ath12k) suspend/resume functionality on the Lenovo Yoga Slim 7x (Snapdragon X Elite).

## Description
Fixes critical bugs introduced by commit `8d5f4da8d70b` that completely broke suspend/resume functionality on Qualcomm Snapdragon X Elite (SC8380XP) platforms with WCN7850 WiFi.

## Exact Issue

### Issue: CMA Page Corruption and Resume Failure on Snapdragon X Elite

**Problem:**
Commit `8d5f4da8d70b` ("wifi: ath12k: support suspend/resume") introduced critical bugs that completely broke suspend/resume functionality on Qualcomm Snapdragon X Elite (SC8380XP) platforms with WCN7850 WiFi:

1. **CMA page corruption during resume** - System crashes with memory corruption when resuming from suspend
2. **MHI state machine failure** - WiFi device fails to reinitialize, leaving WiFi non-functional after resume
3. **Premature system wakeup** - Laptop wakes immediately after entering suspend (deep sleep broken)

**Symptoms:**
- Kernel crashes with "Bad page state" errors during resume
- WiFi dead after resume (cannot reconnect)
- System wakes instantly instead of staying in suspend
- dmesg shows MHI initialization errors: `failed to set mhi state INIT(0) in current mhi state (0x1)`

**Root Cause:**
The original suspend implementation called `mhi_unprepare_after_power_down()` during suspend, which prematurely freed DMA buffers (fbc_image, rddm_image) allocated in CMA. When these freed pages were accessed during resume, the kernel detected corruption. Additionally, the MHI state machine remained in INIT state after suspend, preventing proper reinitialization on resume.

**Solution:**
Skip `ATH12K_MHI_DEINIT` during suspend to preserve DMA buffers and device state, and check if MHI is already initialized before attempting to reinitialize on resume.

**Status:** Submitted to upstream (linux-wireless)

**Testing:** 100+ suspend/resume cycles on Lenovo Yoga Slim 7x (Snapdragon X Elite)

### Kernel Logs

<details>
<summary><b>CMA Corruption Error (Before Fix)</b></summary>

```
[  966.572977]  dm_mirror dm_region_hash dm_log dmi_sysfs autofs4 qrtr aes_neon_bs aes_neon_blk aes_ce_blk aes_ce_cipher
[  966.572994] CPU: 7 UID: 0 PID: 31487 Comm: kworker/u50:7 Kdump: loaded Tainted: G    B               6.19.0-rc7-1ubuntu9-qcom-x1e #1ubuntu9 PREEMPT(voluntary) 
[  966.572998] Tainted: [B]=BAD_PAGE
[  966.573000] Hardware name: LENOVO 83ED/LNVNB161216, BIOS NHCN60WW 09/11/2025
[  966.573002] Workqueue: async async_run_entry_fn
[  966.573007] Call trace:
...
[  966.573046]  dma_free_contiguous+0xd0/0x110
[  966.573053]  dma_direct_free+0x148/0x1e0
[  966.573058]  dma_free_attrs+0xe8/0x220
[  966.573063]  mhi_free_bhie_table+0x50/0xa0 [mhi]
[  966.573074]  mhi_unprepare_after_power_down+0x30/0x70 [mhi]
[  966.573085]  ath12k_mhi_stop+0xf8/0x210 [ath12k]
...
[  966.573300] BUG: Bad page state in process kworker/u50:7  pfn:0xf79ba
[  966.573302] page: refcount:1 mapcount:0 mapping:0000000000000000 index:0x0 pfn:0xf79ba
```
</details>

<details>
<summary><b>Resume Failure Error (Before Fix)</b></summary>

```
[  361.046075] ath12k_wifi7_pci 0004:01:00.0: failed to set mhi state INIT(0) in current mhi state (0x1)
[  361.046085] ath12k_wifi7_pci 0004:01:00.0: failed to set mhi state: INIT(0)
[  361.046090] ath12k_wifi7_pci 0004:01:00.0: failed to start mhi: -22
```
</details>

## Patch Details
- **File**: `0001-wifi-ath12k-fix-CMA-error-and-MHI-state-mismatch.patch`

## Instructions

Apply the patch directly to your kernel source:

```bash
cd /path/to/kernel/source
git am < /path/to/patches/0001-wifi-ath12k-fix-CMA-error-and-MHI-state-mismatch.patch
```

## Tested On
- Kernel Version: **7.0-rc1**
- **Hardware:** Lenovo Yoga Slim 7x (Snapdragon X Elite)
- **WiFi:** WCN7850 hw2.0 PCI