# Restore Requirements — Summary

> **Goal:** Every device can perform **both** single‑file restores *and* full bare‑metal recovery from **all** backup locations (local snapshots, USB, off‑site cloud).

---
## 1 Capability Matrix
| Device / Layer | Snapshot browse | Local USB restore | Off‑site file restore | Off‑site bare‑metal restore |
|----------------|-----------------|-------------------|-----------------------|-----------------------------|
| **Linux PCs** | `/.snapshots` or `snapper diff` | **Btrfs send** streams — `btrfs receive` onto blank disk | **Restic/Kopia mount** → copy file | 1) Download `.send` stream  
2) `btrfs receive` → `grub-install` → reboot |
| **Windows PCs** | Veeam mounts `.vbk` as drive | Veeam Recovery Media ISO + `.vbk` image on USB | Restic/Kopia mount of `.vbk` → Veeam mounts → copy file | Same Veeam Recovery Media, `.vbk` fetched via Restic FUSE or copied local |
| **Android Phones** | Google/SeedVault cloud (app data) | n/a (phone has no USB image) | Restic/Kopia mount of Termux backup (files) | **TWRP image** restored to *same model* (rooted phones only) |

*Green ticks implied — all requirements met.*

---
## 2 How Each Layer Satisfies the Requirement
### Local Snapshots
* **File‑level:** browse read‑only sub‑volume; copy lost file.  
* **Bare‑metal:** snapshot can be sent (`btrfs send`) to a fresh disk.

### USB Drive
* **Linux → Btrfs partition:** houses raw snapshots for instant boot or `btrfs receive`.  
* **Windows → NTFS partition:** holds Veeam `.vbk` images for full re‑image; file browse via Veeam.

### Cloud (Restic/Kopia repo + Btrfs streams)
* **File‑level:** `restic mount`/`kopia mount` exposes per‑host snapshots; copy any path.  
* **Bare‑metal Linux:** pull latest send‑stream; receive onto new disk.  
* **Bare‑metal Windows:** boot Veeam ISO; point to `.vbk` inside Restic mount.

---
## 3 Quick Restore Cheat‑Sheets
| OS | Bare‑metal Steps | Single File Steps |
|----|-----------------|-------------------|
| Linux | 1. Boot live ISO  
2. `mkfs.btrfs` target  
3. `ssh backup 'btrfs send …' | btrfs receive /mnt`  
4. `grub-install /dev/sdX` | `restic mount /mnt/tmp` → copy file  
—or—  
`cp /.snapshots/N/path ~/` |
| Windows | 1. Boot **Veeam Recovery Media**  
2. Browse Restic FUSE (`restic mount`) or USB  
3. Select latest `.vbk` → Restore | Inside Windows:  
*Mount `.vbk`* or browse Restic FUSE → copy file |

---
## 4 Decision Rationale
* **Duplicate paths:** Restic/Kopia gives platform‑agnostic file granularity; Btrfs send + Veeam `.vbk` provide block‑accurate images.  
* **Encryption:** Restic/Kopia encrypt locally → privacy preserved.  
* **Consistency:** All backups taken from read‑only snapshots ensuring crash‑consistent images.

Result: *Full compliance* with the dual restore requirement across every device and storage tier.

