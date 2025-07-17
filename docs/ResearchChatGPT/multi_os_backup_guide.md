# Unified Backup Strategy for Linux **and** Windows PCs

This guide captures the cross‑platform plan discussed: one off‑site location, encrypted repositories, bare‑metal **and** file‑level restores for every family computer.

---
## 1 Why Share an Off‑Site Bucket?
| Benefit | Detail |
|---------|--------|
| **Lower cost** | Storage providers bill by total GB, so pooling saves money. Restic/Kopia deduplicate *across hosts*, storing duplicate ISO/media blocks once. |
| **Simpler admin** | One credential, one monitoring dashboard, one retention policy. |
| **Consistent tooling** | Same backup commands on every machine; fewer restore procedures to remember. |

---
## 2 Engine Options & Cross‑Host Dedup
| Tool | Runs on | Multi‑host dedup? | Note |
|------|---------|------------------|------|
| **Restic** | Linux, Windows, macOS | **Yes** (host tag) |  Single binary, S3‑native |
| **Kopia** | Linux, Windows, macOS | **Yes** | GUI + CLI, fast prune |
| **Borg 1.x** | Linux & *via* WSL on Win | No (one host per repo) | Borg 2 roadmap adds multi‑host but not stable yet |
| **Duplicati / UrBackup** | All major OSes | Yes | Heavier SQL backend |

**Recommendation:** use **Restic or Kopia** as the cross‑platform engine.

---
## 3 Keeping Btrfs `send/receive` *and* Windows Backups in One Account
```
remote/
 ├─ restic-repo/              ← encrypted: Windows & Linux backups
 └─ btrfs-snapshots/          ← raw send‑receive streams (Linux only)
     ├─ laptop‑aeon/
     └─ desktop‑tw/
```
* Linux hosts push both:
  * `btrfs send` snapshot streams to `btrfs-snapshots/…` *(bit‑perfect history)*
  * Same snapshot path to **Restic** for file‑level restore/dedup.
* Windows hosts push **Restic** backups only (see §4 imaging tools).

Single **SSH** or **S3** credential covers both trees.

---
## 4 Bare‑Metal Imaging on Windows
| Tool | File‑level browse | Full‑disk restore | Licence status | Why it fits |
|------|------------------|-------------------|----------------|-------------|
| **Veeam Agent Free** | ✔ mounts `.vbk` | ✔ Recovery Media ISO | Actively supported | Images are single large files → Restic handles them easily |
| Rescuezilla / Clonezilla | ✔ | ✔ | Open‑source | Slower incrementals |
| Macrium Reflect Free | ✔ | ✔ | *Retired 2024* | New installs require paid licence |

**Chosen path:** install **Veeam Agent Free** on each Windows PC, target the USB drive (NTFS side) and let Restic upload the `.vbk` files.

---
## 5 Daily Workflows
### Linux Snapshot + Dual Push
```bash
# hourly.timer
btrfs sub snap -r /home /.snapshots/`date +%s`
# Send to off‑site Btrfs tree
btrfs send -p LAST /.snapshots/$(date +%s) | \
  ssh backupbox "btrfs receive /remote/btrfs-snapshots/$(hostname)"
# File‑level copy into Restic repo
restic -r /remote/restic-repo backup /.snapshots/$(date +%s) --tag $(hostname)
```

### Windows Task Scheduler Snippet
1. Nightly Veeam incremental → `D:\Backups`.  
2. Post‑job script:
```powershell
restic.exe -r X:\remote\restic-repo backup D:\Backups\*.vbk --host WIN-LAPTOP
```

---
## 6 Restore Cheat‑Sheets
| Scenario | Steps |
|----------|-------|
| **Linux bare‑metal (disk died)** | Boot live ISO → `mkfs.btrfs /dev/nvme0n1` → `mount /dev/nvme0n1 /mnt` → `ssh backupbox 'btrfs send /remote/btrfs-snapshots/laptop/latest' | btrfs receive /mnt` → `grub-install` → reboot |
| **Linux single file** | `restic -r /remote/restic-repo mount /mnt/tmp` → copy file from `/mnt/tmp/snapshots/LAPTOP/...` |
| **Windows bare‑metal** | Boot Veeam Recovery Media → point to `.vbk` via Restic FUSE (`restic mount`) or copy locally → wizard restores image |
| **Windows single file** | Mount `.vbk` in Veeam or browse Restic mount and copy needed file |

---
## 7 Cost & Administration Summary
| Layer | Tool / Provider | Cost (1 TB) | Hands‑on time |
|-------|-----------------|-------------|---------------|
| Off‑site | Hetzner Storage Box **or** Backblaze B2 | €3.20 / mo (SSH) or $6 / TB‑mo (S3) | Key rotation & retention tweaks quarterly |
| Local USB | Btrfs + NTFS split | sunk cost | Scrub + SMART check monthly |
| Monitoring | `borgmatic check`, `restic check`, or Kopia’s UI | negligible | Weekly cron + email |

---
## 8 TL;DR Workflow
1. **Restic or Kopia** is the single cross‑platform repository engine.  
2. Linux adds **Btrfs send** for block‑perfect history, stored alongside the Restic repository in the same account.  
3. Windows uses **Veeam Agent Free** → Restic.  
4. One credential, one bucket; both file‑level and bare‑metal restores covered for every device in the house.

