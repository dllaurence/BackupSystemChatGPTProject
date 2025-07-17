# Implementation Plan (D‑25)

This plan satisfies every **Implementation Requirement** and drives the project toward Milestones M‑1 … M‑6.  Each phase is atomic and reversible.

---
## Milestones

| ID | Description | Achieved when… |
|----|-------------|----------------|
| **M‑1** | Backup of Fedora 40 | Snapshots + NTFS USB Restic backup succeed and test restore works. |
| **M‑2** | End‑to‑end backups on Fedora 41 | Snapshots, Btrfs USB backup, weekly cloud upload validated. |
| **M‑3** | openSUSE Tumbleweed running | TW boots via GRUB, shares `/home`, passes smoke tests. |
| **M‑4** | End‑to‑end backups on Tumbleweed | Snapper snapshots replicate to USB & cloud; restores verified. |
| **M‑5** | Windows backups integrated | Veeam images stored on USB + cloud; test bare‑metal restore documented. |
| **M‑6** | Android backups integrated | SeedVault/Termux Restic backups stored locally & off‑site; file restore verified. |

---
## Phase 0  Project bootstrap

| Task | Commands / notes |
|------|------------------|
| 0‑1 | Create/clone private GitHub repo `backup-system`; push current docs. |
| 0‑2 | Add folders: `scripts/`, `systemd/`, `hooks/`, `docs/`. |
| 0‑3 | Install `pandoc` for later polished PDFs (optional). |

---
## Phase 1  Snapshots on Fedora 40  → *M‑1 step 1*

| Task | Detail |
|------|--------|
| 1‑1 | `sudo dnf install btrbk` |
| 1‑2 | Add minimal `/etc/btrbk/btrbk.conf` (hourly, snapshot_dir `.snapshots`). Commit to Git. |
| 1‑3 | `systemctl enable --now btrbk-hourly.timer` |
| 1‑4 | Verify after 2 h: `btrfs subvolume list / | grep snapshots`. |
| 1‑5 | Commit `systemd/btrbk-hourly.timer` & `.service` to repo. |

---
## Phase 2  USB NTFS backup via Restic   → *M‑1 completed*

| Task | Detail |
|------|--------|
| 2‑1 | Plug 4.5 TB USB; confirm NTFS slice `WINBKUP` mounted. |
| 2‑2 | `restic init ‑‑repo /run/media/$USER/WINBKUP/restic-repo` |
| 2‑3 | Create `scripts/usb_backup.sh`: `restic backup /.snapshots --tag NTFSUSB` |
| 2‑4 | `systemd-run --unit=usb-backup --on-calendar='hourly' scripts/usb_backup.sh` |
| 2‑5 | Verify `restic snapshots`. Commit script + generated unit files. |
| 2‑6 | **Milestone M‑1 achieved.** |

---
## Phase 3  Partition surgery & Fedora 41 fresh install

| Task | Detail |
|------|--------|
| 3‑1 | Live USB: shrink F40 Btrfs by 150 GB; create ext4 `/boot` (1 GiB) + ESP‑B (512 MiB). |
| 3‑2 | Create Btrfs subvolume `@rootF41`. |
| 3‑3 | Install Fedora 41 into `@rootF41`, mounting existing `/home`, `/boot`, both ESPs. |
| 3‑4 | Post‑install: add identical `resume=` UUID to kernel cmdline. |
| 3‑5 | Boot F41; install `btrbk`; copy snapshot timer config (temporary). |

---
## Phase 4  USB reformat + cloud setup   → *M‑2 completed*

| Task | Detail |
|------|--------|
| 4‑1 | Repartition USB to 1 TB NTFS / 500 GB cushion / ~3 TB Btrfs. |
| 4‑2 | `btrfs send` replication: enable `btrbk-backup.timer` targeting `LINBKUP`. |
| 4‑3 | Cloud: `restic -r wasabi:s3… init`; weekly `cloud-backup.timer`. |
| 4‑4 | Test restore from cloud (`restic restore --target /tmp/restore`). |
| 4‑5 | Disable temporary NTFS restic timer. |
| 4‑6 | **Milestone M‑2 achieved.** |

---
## Phase 5  Install openSUSE Tumbleweed   → *M‑3*

| Task | Detail |
|------|--------|
| 5‑1 | Create subvolume `@rootTW`; run TW installer in expert partitioning. |
| 5‑2 | TW installer sets up **Snapper** by default; leave enabled. |
| 5‑3 | Copy `btrbk` config; ensure Snapper snapshots excluded from duplicate creation. |
| 5‑4 | Test dual‑boot; verify ESP sync hook updates TW kernels. |
| 5‑5 | Snapshot restore smoke‑test on TW. |
| 5‑6 | **Milestone M‑3 achieved.** |

---
## Phase 6  Finalise Linux backups on Tumbleweed   → *M‑4*

| Task | Detail |
|------|--------|
| 6‑1 | Ensure `btrbk` replicates Snapper snapshots (`snapshot_dir .snapshots`). |
| 6‑2 | Cloud weekly job now runs from TW root; disable F41 jobs. |
| 6‑3 | Delete Fedora 40 root if no longer needed; keep F41 as test root. |
| 6‑4 | Full restore test from USB onto spare NVMe (lab). |
| 6‑5 | **Milestone M‑4 achieved.** |

---
## Phase 7  Integrate Windows backups   → *M‑5*

| Task | Detail |
|------|--------|
| 7‑1 | Install Veeam Agent Free on Windows PCs; schedule nightly jobs to `\WINBKUP`. |
| 7‑2 | Post‑job script calls `restic backup` on latest `.vbk`. |
| 7‑3 | Verify Restic snapshots include Windows host tags. |
| 7‑4 | Document bare‑metal recovery using Veeam ISO + Restic mount. |
| 7‑5 | **Milestone M‑5 achieved.** |

---
## Phase 8  Integrate Android backups   → *M‑6*

| Task | Detail |
|------|--------|
| 8‑1 | For stock phones: install Termux + `pkg install restic`; cron job backs up `/sdcard`. |
| 8‑2 | For custom ROM phones: configure SeedVault SFTP to `LINPOOL:/seedvault/PHONE`. |
| 8‑3 | Include `/seedvault` path in btrbk send and Restic backups. |
| 8‑4 | Perform file‑level restore test and (if rooted) full IMG restore. |
| 8‑5 | **Milestone M‑6 achieved.** |

---
## Appendix A  Temporary‑tool markers

| Temp tool | Phase | Removal |
|-----------|-------|---------|
| btrbk snapshot on F41 (while Snapper not yet active) | 3 | Removed in Phase 6 |
| Restic NTFS timer | 2 | Removed in Phase 4 |

All temporary tools are labelled *TEMP* in their unit filenames and must be deleted before project completion.
