# Backing Up Android Devices to the Unified Off‑Site Repository

This note distills the earlier phone‑backup discussion into a single reference you can drop into the repo.

---
## 1  Backup Methods by Phone Type
| Method | Root / unlock? | Captures | How it plugs into **Restic/Kopia repo** |
|--------|----------------|----------|-----------------------------------------|
| **Termux + Restic / Kopia** | ✕ | Any folder reachable in `/sdcard` (DCIM, Download, WhatsApp, etc.) | `restic backup /sdcard/DCIM … --host PHONE‑PIXEL8` (run via Termux:Boot or Tasker). Uploads straight to the shared repo. |
| **SeedVault** (CalyxOS, LineageOS, /e/OS) | ✕ | App APK + data, SMS, call log, user‑selected dirs | SeedVault’s *SFTP* destination points at `remote:/seedvault-phone/`; a PC cron job then `restic backup` that folder. |
| **TWRP / Nandroid IMG** | √ | Block‑level image of system, vendor, data, boot | Copy IMG folder to a PC or USB; `restic backup /TWRP/BACKUPS/... --tag NANDROID`. |
| **Syncthing Android** | ✕ | Real‑time mirror of selected folders | PC receives folder → PC’s snapshot‑to‑Restic timer picks it up. |

---
## 2  Restore Paths
| Scenario | Restore Steps |
|----------|---------------|
| **Lost photo / doc** | `restic mount` on any PC → browse to `/snapshots/PHONE-*/<path>` → copy. |
| **New phone (same Google acct)** | Google Cloud restores contacts, SMS, settings *automatically*; Termux‑Restic job restores media after first sync. |
| **New phone (SeedVault ROM)** | SeedVault “Restore apps & data” from same SFTP path. |
| **Full rollback after modding mishap** | Boot TWRP → push latest IMG from Restic mount to `/sdcard/TWRP/BACKUPS/...` → “Restore”. |

---
## 3  Scheduling & Bandwidth
| Job | Frequency | Data volume |
|-----|-----------|-------------|
| Termux‑Restic file backup | Daily (Termux:Boot cron) | First run 15–20 GB; nightly delta few hundred MB. |
| SeedVault SFTP push | Nightly (built‑in) | App data ZIPs; tens of MB. |
| Restic prune | Weekly on repo host | Keeps 7 daily / 4 weekly / 6 monthly snapshots. |

Backblaze B2 cost ≈ $1.20/month for one 128 GB phone; virtually free on a Hetzner Storage Box (fits under 1 TB).  

---
## 4  Privacy
* All phone data arrives **already encrypted** (Restic/Kopia AES‑256, SeedVault AES+ChaCha).  
* Cloud provider never sees plaintext; same encryption keys as PC backups.  

---
## 5  TL;DR Implementation
1. **Non‑root stock phones:** install Termux, Restic binary, schedule daily backup of `/sdcard`.  
2. **Custom ROM phones:** enable SeedVault → SFTP to the same remote host.  
3. **Rooted tinkerer phones:** add periodic TWRP images; store via Restic.  
4. No extra cloud accounts — everything lands in the **same encrypted repository** you already pay for.

