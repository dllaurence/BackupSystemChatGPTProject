# Backup System Requirements (Revision 13)

## Must‑have
1. **Prefer best‑tested, proven tools & techniques** — default choice unless documented reason to diverge.  
2. Automatic local snapshots (hourly/daily).  
3. Off‑site copy (second USB or strongly‑encrypted cloud) for full 3‑2‑1 coverage.  
4. Btrfs subvolumes for logical separation (`@rootA`, `@rootB`, `@home`, `@snapshots`).  
5. Boot continuity via the factory ESP on Disk 0.  
6. Redundant ESP on Disk 1 kept in sync automatically.  
7. Incremental send/receive replication to the USB backup disk.  
8. Clear retention policies on both the internal and backup disks.  
9. Automatic tolerance & catch‑up for the USB disk (queue while away, flush on return).  
10. Single shared 64 GB swap partition with safe `resume=` handling.  
11. Monthly `btrfs scrub` with alerting.  
12. Strong encryption for **all** backup targets.  
13. Automatic health monitoring / push alerts (mail, SMS, etc.).  
14. GitHub repository to store **all** docs, scripts, and timers.  
15. **Cross‑platform coverage** — Windows bare‑metal images & file‑level restore; Android device backups feeding the same encrypted off‑site repo.  
16. **Restore guarantees** — every tier (snapshots, USB, cloud) must support both single‑file and full bare‑metal restores for Linux, Windows, and Android.  
17. **Provider‑flexible cost tracking** — design must allow cloud‑provider swap with maintained cost tables (Backblaze B2, Wasabi, iDrive e2, Hetzner Box, etc.).  
18. **Windows built‑in backups are supplementary only** — formal plan cannot rely on OneDrive/File History; Veeam Agent Free (or equivalent) provides official imaging.  
19. **Documentation upkeep** — “Multi‑OS backup guide” & “Restore capabilities summary” must stay synced with implementation.

## Should‑have
1. **Minimal configuration in the *final* design** — use distro defaults when they satisfy specs; temporary deviations in mock‑ups are allowed and must be labelled.  
2. Per‑subvolume quotas (qgroups).  
3. Swing‑space layout on the backup disk.  
4. Convertible encryption path for the internal Linux disks.  
5. Multi‑machine sharing safeguards (per‑host subvolumes & retention).  
6. Inexpensive providers that still meet the strong‑encryption rule.  
7. **Version‑skew management on shared `/home`** — policies to avoid config corruption when multiple roots share the same home.

## Nice‑to‑have
1. Quarterly maintenance checklist.  
2. Tool‑set documentation (`tool‑set.md`).  
3. Provider anonymity / privacy extras.  
4. GUI rollback tool **(prefer Qt over GTK)**.  
5. Automated swing‑space reclaim script.  
6. Design notes include how well‑tested / battle‑proven each technique is.

---

## Distro Choice Requirements
* **Primary production Linux**: openSUSE Tumbleweed (or Aeon variant) for Secure‑Boot and NVIDIA friendliness.  
* **Test / secondary slots**: Fedora, Arch‑based, Nobara, Siduction as space allows, installed into separate Btrfs subvolumes or root partitions while sharing `/boot`, ESPs, and `/home`.  
* Each distro must tolerate the shared ext4 `/boot` and redundant ESP layout.

---

## Implementation Clarifications
* “Minimal configuration” applies only to the final stabilized system. Mock‑ups may employ extra tools or simpler defaults if that accelerates milestones or learning; such deviations must be flagged *temporary* in the Implementation Plan and removed before Milestone M‑4.
