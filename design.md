# Backup System – Architecture Design **v3**  
*(Aligned with Requirements rev 13 – cross‑platform, provider‑flexible, multi‑distro)*

---
## 0 Scope Overview
| Layer | Linux (Atlantis) | Windows PCs | Android Phones |
|-------|------------------|-------------|----------------|
| Local snapshots | Btrfs Snapper/btrbk | VSS inside Veeam image | SeedVault / Termux snapshot dir |
| Local USB backup | Btrfs send (LINBKUP) | Veeam `.vbk` to NTFS slice | Optional copy of SeedVault SFTP |
| Off‑site backup | Restic repo (+ raw send dir) | Same Restic repo | Same Restic repo |
| Restore granularity | File & bare‑metal | File & bare‑metal | File, and full IMG (rooted) |

Single encrypted Restic repository deduplicates across **all devices**; Btrfs send streams and `.vbk` images stored alongside for block‑level restorations.

---
## 1 Disk & Partition Layout (Atlantis)

| Disk | Part | Size | FS | Mount | Purpose |
|------|------|------|----|-------|---------|
| Disk 0 (512 GB) | p1 | 512 MiB | FAT32 | **/boot/efi** | Factory ESP |
|                | p2 | ~rest | NTFS | — | Windows C: |
| Disk 1 (2 TB)  | p1 | 512 MiB | FAT32 | **/boot/efi2** | Redundant ESP |
|                | p2 | 1 GiB  | ext4 | **/boot** | Shared boot slice (all distros) |
|                | p3 | 64 GB  | swap | — | Single swap |
|                | p4 | rest  | Btrfs `LINPOOL` | `/` (subvolumes) | Main pool |

USB **4 .5 TB** external drive:

| Part | Size | FS | Label | Role |
|------|------|----|-------|------|
| 1 | **1 TB** | NTFS | **WINBKUP** | Veeam images + Windows File History |
| 2 | **500 GB** | ext4 | **CUSHION** | swing‑space |
| 3 | **≈ 3 TB** | Btrfs | **LINBKUP** | Btrfs send/receive target + Restic repo cache |

---
## 2 Subvolume Tree (Linux)

```
LINPOOL
 ├─ @rootTW        # openSUSE Tumbleweed (prod)
 ├─ @rootF41       # Fedora 41 (test)
 ├─ @rootNobara    # optional gaming root
 ├─ @home
 ├─ @snapshots     # read‑only snapshots
 └─ Atlantis‑meta  # scripts / docs
```

Quotas off by default; enable if `/home` >85 % usage (see Quarterly checklist).

---
## 3 Snapshot & Replication Flow (Linux)

```
Snapper (TW) / btrbk (others)  →  @snapshots (hourly/daily)
           |                       |
           +--> btrbk send → USB:Btrfs  (udev‑trigger)
           |       retention 12m/3y
           |
           +--> Restic backup tag=LINUX → cloud (weekly)
```

*Cloud target defaults to **Wasabi**; Backblaze B2 and iDrive e2 configs kept in `configs/providers/`.*

---
## 4 Windows Backup Flow

1. **Veeam Agent Free** nightly incremental → `E:\Backups\PC‑NAME.vbk` (on WINBKUP NTFS).  
2. Post‑job PowerShell:
   ```powershell
   restic.exe -r s3:https://s3.wasabisys.com/mybucket backup E:\Backups\PC-*.vbk --host WIN-DESKTOP
   ```
3. `.vbk` dedupes in Restic; USB copy serves as fast local restore.

Restores: Veeam Recovery Media ISO → Restic mount (FUSE) or USB path.

---
## 5 Android Backup Flow

* **Stock phone:** Termux + Restic daily cron backs up `/sdcard` → Restic repo (`--host PHONE‑PIXEL8`).  
* **Custom ROM:** SeedVault nightly SFTP to `LINPOOL:/seedvault/PHONE` → included in btrbk send + Restic.  
* **Rooted tinkerer:** monthly TWRP image copied to `/TWRP` then picked up by Restic.

Off‑site dedup compresses media across phones & PCs.

---
## 6 Off‑Site Repository Structure (Wasabi or alternate)

```
s3://mybucket/
 ├─ restic-repo/         # encrypted blobs, all hosts
 └─ raw-snapshots/       # optional btrfs send streams
      ├─ atlantis/
      └─ laptop-tw/
```

Provider swap: change remote URL + credentials in `restic.env`; no other code changes.

Cost table maintained in `docs/backup_plan.md` (Backblaze, Wasabi, iDrive, Hetzner).

---
## 7 ESP Sync & Boot Redundancy

* Both ESPs mounted (`/boot/efi`, `/boot/efi2`).  
* Post‑kernel‑update hook `20-sync-esp.sh` → `rsync -a --delete`.  
* UEFI BootOrder: ESP0 then ESP1.  
* **Ext4 /boot** satisfies Fedora, Arch, Siduction installers.

---
## 8 Notifications & Health Monitoring

| Event | Method | Tool |
|-------|--------|------|
| Snapshot/backup failure | E‑mail + SMS gateway | `backup-alert@.service` |
| USB stale > 7 days | Same alert | `btrbk-health.timer` |
| Cloud prune error | Matrix (optional) | `restic-check@.service` |

Scripts stored in `scripts/alerts/`; templated via systemd `Environment=` vars.

---
## 9 Restore Guarantees Matrix

| Device | Local snapshot | USB | Cloud |
|--------|----------------|-----|-------|
| Linux | copy file or `btrfs receive` | `btrfs receive` | Restic mount **or** Btrfs stream |
| Windows | VSS in `.vbk` | Veeam ISO + `.vbk` | Veeam ISO + Restic‐mounted `.vbk` |
| Android | SeedVault / file copy | (n/a) | Restic mount; TWRP IMG (rooted) |

See `docs/restore_capabilities_summary.md` for cheat‑sheets.

---
## 10 Tool Set & Provenance

| Role | Primary tool | Proven maturity |
|------|-------------|-----------------|
| Linux snapshot | **Snapper** (TW) / **btrbk** (others) | SUSE default since 2012 / btrbk 1.10 LTS |
| Replication | **btrbk** | Widely used in Arch & Gentoo communities |
| Windows imaging | **Veeam Agent Free** | Enterprise vendor; > 1 M users |
| Cross‑platform repo | **Restic 0.16** | Active since 2016; reproducible builds |
| Encryption | **LUKS2**, Restic AES‑256 | Kernel native / audited |
| ESP sync | `rsync` hook | Ubuntu & SUSE docs example |

Each choice either ships as distro default or has multi‑year production history → satisfies **“best‑tested”** requirement.

---
## 11 Provider‑Flexible Cost Tracking

Located in `docs/backup_plan.md` – table auto‑updated quarterly (script `scripts/update_costs.py`).

---
## 12 Version‑Skew Management on Shared `/home`

* TW & Aeon kept <2 day skew (auto timers).  
* Snapshot before booting a test distro; mount `/home` read‑only first boot.  
* UID consistency enforced (`useradd -u 1000`).  
* Guide: `docs/skew_guide.md`.

---
## 13 Milestones & Phases

| Milestone | Key deliverables |
|-----------|------------------|
| **M‑1** Backup running on Fedora 40 (snapshots + USB NTFS via Restic) |
| **M‑2** End‑to‑end backups on Fedora 41 (Btrfs USB + cloud) |
| **M‑3** openSUSE Tumbleweed installed (dual‑boot) |
| **M‑4** End‑to‑end backups on Tumbleweed; Windows + Android integrated |

Implementation phases map 1:1 to these milestones (see `docs/implementation_plan.md`).

---
## 14 Open Issues / Future Enhancements

* Evaluate **Kopia UI** once Linux & Windows repos exceed 5 TB.  
* Consider **Hetzner Storage Box** if Wasabi costs rise.  
* NAS pull‑through cache for faster local restores.

---
## TL;DR

* Linux roots on Btrfs with Snapper/btrbk snapshots → USB Btrfs send + Restic cloud.  
* Windows PCs image with Veeam, uploaded via Restic; Android phones push via Termux or SeedVault.  
* One encrypted S3 bucket stores everything, deduplicated, provider‑agnostic.  
* Redundant ESPs + shared ext4 `/boot` keep multi‑distro boots safe.  
* All choices are defaults or long‑running community staples, fulfilling the “proven tools” mandate.
