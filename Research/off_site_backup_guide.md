# Off‑Site Backup Guide

This document captures the engine choices, provider comparisons, and reference workflows for extending your local USB + snapshot strategy into a full **3‑2‑1** scheme (3 copies, 2 media, 1 off‑site) — without buying a NAS right away.

---
## 1 Engine Options for Cloud Backups
| Engine | What it gives you | Ideal when | Snapshot workflow |
|--------|------------------|------------|-------------------|
| **Restic / Kopia** (single‑binary, S3‑native) | Fast multi‑thread upload, built‑in AES‑256 encryption, compression & dedup; runs on Linux *and* Windows | You want one tool everywhere and object‑storage pricing | `btrfs sub snap -r /home …` → `restic backup` snapshot path; each backup is its **own point‑in‑time** in Restic/Kopia |
| **Borg + Borgmatic** (SSH target) | Rock‑solid; FUSE browse (`borg mount`); append‑only mode | You prefer turnkey **backup host** (rsync.net, BorgBase, Hetzner Storage Box) instead of raw object storage | Same snapshot trick; duplicate blocks deduped |
| **`btrfs send | receive` over SSH** | Bit‑for‑bit clone of every snapshot (reflinks, ACLs, compression) | Remote target is **also Btrfs** (cheap VPS, Hetzner box) and you’re happy managing raw sub‑volumes | `btrfs send -p LAST … | ssh host btrfs receive …` keeps **entire snapshot tree** |

> *Why not pipe Btrfs streams into S3?* Each stream is one huge file — you’d lose server‑side dedup and pay full egress to restore it.

---
## 2 S3‑Compatible Object‑Storage Providers
| Provider | Base cost | Egress/API fees | Pros / Cons |
|----------|-----------|-----------------|--------------|
| **Backblaze B2** | **$6 / TB‑month**; first 10 GB free | 3× stored data egress free, then $0.01/GB | Long track‑record; Restic backend; cheapest total if you test restores |
| **Wasabi Hot Cloud** | **$6.99 / TB‑month**, no egress; 90‑day min retention | none | Predictable bill; multiple regions; great if you worry about restore costs |
| **iDrive e2** | $0.005/GB (first TB ≈ $5) | $0.01/GB egress | Cheapest <1 TB bucket; younger service, but mature parent company |
| **Storj (decentralised)** | $4 / TB‑month + $5 minimum use | $7 / TB egress | End‑to‑end encrypted by design; slower first‑byte latency |

---
## 3 Hosted‑SSH Boxes for Borg or Btrfs Send
| Service | Price (~1 TB) | Traffic fees | Highlights |
|---------|---------------|--------------|------------|
| **Hetzner Storage Box BX11** | **€3.20 / mo** (~$3.50) | none | SSH/WebDAV/SFTP; Borg & Restic presets; 10 automatic snapshots |
| **rsync.net Borg account** | **$0.008 / GB‑mo** (≈ $8/TB) | none | ZFS backend with daily 30‑day snapshots; immutability flag |
| **BorgBase** | $2 / mo (10 GB) → $80 / yr (1 TB) | none | 10 GB free tier; web dashboard; stale‑backup alerts |

---
## 4 Reference Workflows
| Goal | Tool stack | Target | Approx. cost |
|------|-----------|--------|--------------|
| **Maximum automation, zero surprises** | Restic (AES‑256) + daily systemd timer against snapshot | **Wasabi** bucket | ≈ $7 / TB‑month; no egress fees |
| **Cheapest TB price with Borg comfort** | Borgmatic (prune & health) against snapshot | **Hetzner Storage Box** | €3.20 / mo per TB; unlimited traffic |
| **Full snapshot tree off‑site** | `btrbk` incremental `btrfs send` | **Small Btrfs VPS** (e.g. €2 VPS) | ≈ €5 / mo total |

---
## 5 Future NAS Consideration
A NAS can later:
* Pull hourly snapshots over LAN (laptop offline sooner).  
* Mirror to the **same** cloud bucket with Rclone/Restic.  
* Serve as on‑prem restore cache.

No data reshuffling required — just point the NAS at the existing repo or receive streams.

---
## 6 Quick Decision Grid
| Scenario | Best Choice | Rationale |
|----------|-------------|-----------|
| **Cross‑platform backups (Windows + Linux)** | **Restic/Kopia → Backblaze B2** | Single tool, global dedup, native Windows binary |
| **Linux‑only, budget priority** | **Borgmatic → Hetzner Box** | Lowest €/TB, SSH simplicity |
| **Need perfect Btrfs snapshots** | **`btrfs send` → Hetzner Box (Btrfs)** | Preserves reflinks, ACLs, compression |

---
### TL;DR
* Pick **one engine**: Restic/Kopia for S3, Borg for SSH, or raw Btrfs send.  
* Back‑up **snapshots**, not live mounts.  
* Cheap, trusted picks: Hetzner Box (SSH) or Backblaze B2 (S3).  
* Encryption happens **before** upload, so provider trust is mainly about uptime + billing.

