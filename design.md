# Backup System – Architecture Design v2.1

*(Matches Priority List rev 11 and latest deltas)*

---
## 1 Disk & Partition Layout

| Disk | Part | Size | FS | Mount | Purpose |
|------|------|------|----|-------|---------|
| Disk 0 (512 GB) | p1 | 512 MiB | FAT32 | **/boot/efi** | Factory ESP |
|                | p2 | ~rest | NTFS | — | Windows C: |
| Disk 1 (2 TB)  | p1 | 512 MiB | FAT32 | **/boot/efi2** | Redundant ESP |
|                | p2 | 1 GiB  | ext4 | **/boot** | Shared boot slice |
|                | p3 | 64 GB  | swap | — | Single swap |
|                | p4 | rest  | Btrfs `LINPOOL` | `/` (subvolumes) | Main pool |

---
## 2 Subvolume Tree

```
LINPOOL
 ├─ @rootA        (/)
 ├─ @rootB        (alternate distro)
 ├─ @home         (/home)
 ├─ @snapshots    (read‑only)
 └─ Atlantis-meta (scripts/docs)
```

Quotas **disabled by default**; enable only if subvolumes contend for space.

---
## 3 Snapshot & Replication Flow

```
Snapper (openSUSE) / btrbk snapshot  →  @snapshots
        | hourly/daily                  |
        +-------------------------------+
        | btrbk send → USB (udev‑trigger)  |
        | retention: 12m/3y               |
        +---------------------------------+
        | restic + Wasabi (weekly)        |
        | retention: 52w                 |
```

* USB encrypted with LUKS2.  
* Cloud encrypted end‑to‑end (restic).

---
## 4 ESP Synchronisation

Mounted ESPs: `/boot/efi` and `/boot/efi2`  
Kernel post‑transaction hook:

```bash
rsync -a --delete /boot/efi/EFI/Fedora/ /boot/efi2/EFI/Fedora/
```

UEFI `BootOrder`: primary → Disk 0 ESP, fallback → Disk 1 ESP.

---
## 5 Systemd Timers (cron replacement)

| Timer | Service | Function |
|-------|---------|----------|
| `btrbk-hourly.timer` | `btrbk-hourly.service` | Create local snapshot. |
| `btrbk-backup.timer` | `btrbk-backup.service` | Send incrementals to USB/cloud. |
| `btrfs-scrub.timer`  | system service | Monthly scrub, alert on error. |

Timers provide logging via `journald`, calendar syntax, and proper ordering.

---
## 6 Cloud‑Frequency Cost Matrix

| Frequency | Provider | Stored GB | $/mo |
|-----------|----------|-----------|------|
| Monthly | Backblaze B2 | 200 | \$1.00 |
| Monthly | Wasabi | 200 | \$1.40 |
| Weekly *(default)* | Wasabi | 800 | \$5.59 |
| Weekly | Backblaze B2 | 800 | \$4.00 |
| Daily | Backblaze B2 | 6 000 | \$30.00 |
| Daily | Wasabi | 6 000 | \$41.94 |

---
## 7 Notifications

* **Mail** via Namecheap SMTP (default).  
* **SMS** via carrier e‑mail‑to‑SMS gateway (optional).  
* Hooks for Matrix/Pushover documented.

---
## 8 Maintenance & Trip Checklists

* Quarterly: free‑space check, enable quotas if > 85 % used.  
* Trip checklist: ad‑hoc `btrbk snapshot` & `restic backup` before travel.

---
## 9 Tool Set

| Role | Tool (default) |
|------|---------------|
| Snapshot (openSUSE) | Snapper |
| Snapshot (other distros) | btrbk |
| Replication | btrbk |
| Off‑site backup | restic + Wasabi |
| ESP sync | rsync hook |
| Encryption | cryptsetup/LUKS2 |

All choices are **battle‑tested** or distro defaults unless otherwise noted.
