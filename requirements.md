# Backup System Requirements (Revision 12)

## Must‑have
1. **Prefer best‑tested, proven tools & techniques** — default choice unless documented reason to diverge.  
2. Automatic local snapshots (hourly/daily).  
3. Off‑site copy (second USB or strongly‑encrypted cloud) for full 3‑2‑1 coverage.  
4. Btrfs subvolumes for logical separation (`@rootA`, `@rootB`, `@home`, `@snapshots`).  
5. Boot continuity via the factory ESP on Disk 0.  
6. Redundant ESP on Disk 1 kept in sync automatically.  
7. Incremental send/receive replication to the USB backup disk.  
8. Clear retention policies on both the internal and backup disks.  
9. Automatic tolerance & catch‑up for the USB disk (queue while away, flush on return).  
10. Single shared 64 GB swap partition with safe `resume=` handling.  
11. Monthly `btrfs scrub` with alerting.  
12. Strong encryption for **all** backup targets.  
13. Automatic health monitoring / push alerts (mail, SMS, etc.).  
14. GitHub repository to store **all** docs, scripts, and timers.  

## Should‑have
1. **Minimal configuration** — use distro defaults when they satisfy specs; document every deviation.  
2. Per‑subvolume quotas (qgroups).  
3. Swing‑space layout on the backup disk.  
4. Convertible encryption path for the internal Linux disks.  
5. Multi‑machine sharing safeguards (per‑host subvolumes & retention).  
6. Inexpensive providers that still meet the strong‑encryption rule.  

## Nice‑to‑have
1. Quarterly maintenance checklist.  
2. Tool‑set documentation (`tool-set.md`).  
3. Provider anonymity / privacy extras.  
4. GUI rollback tool **(prefer Qt over GTK)**.  
5. Automated swing‑space reclaim script.  
6. Design notes include how well‑tested / battle‑proven each technique is.  

---
## Implementation Requirements

### General
1. **Early subsystem testing** — implement and test each major component (snapshots, USB backup, cloud sync) as soon as feasible so design flaws surface early.  
2. **Small, independent steps** — break work into atomic tasks to avoid leaving Atlantis in a half‑working state; laptop must stay usable for day‑to‑day chores.

### Situation‑specific (Fedora 40 → Fedora 41 transition)
1. **Automatic snapshots on current Fedora 40 Btrfs root** — use btrbk timers (or Snapper if simpler) so you gain hands‑on practice before disk changes.  
2. **Temporary automatic backups to existing NTFS USB drive** — prove the backup flow even on a non‑Btrfs target; tool choice documented in the implementation plan.  
3. **Upgrade to Fedora 41 once backups validated** — proceed only when at least one successful snapshot & USB backup cycle has completed.  
4. **Enable automatic encrypted uploads to cloud (Wasabi default)** — start with weekly cadence; document how to trigger an immediate upload before trips or risky travel.  

These requirements merge operational safety with learning goals and will drive the step‑by‑step Implementation Plan.
