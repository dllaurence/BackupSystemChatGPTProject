# Backup‑System Deliverables (updated)

| ID | Deliverable | Description |
|----|-------------|-------------|
| **D‑01** | Partition‑layout reference | Diagram/PDF of both disks: device names, sizes, UUIDs, mountpoints, swap. |
| **D‑02** | Tutorial | Narrative walk‑through of Btrfs concepts, snapshot flow, backup workflow. |
| **D‑03** | Reference guide | Exhaustive command catalog (`btrfs`, `btrbk`, systemd units), quota syntax, timers. |
| **D‑04** | Quick‑reference sheet | Two‑page crib sheet: common tasks, restore snippets, growth commands. |
| **D‑05** | Backup topology & retention matrix | Snapshot frequencies, keep policies, cloud‑frequency cost table. |
| **D‑06** | Recovery run‑book | Bare‑metal & point‑in‑time restore procedures for all OSes. |
| **D‑07** | Subvolume & quota initializer (`init_btrfs.sh`) | Idempotent script: creates subvols, enables quotas. |
| **D‑08** | Snapshot timer units | `btrbk-hourly`, `btrbk-backup`, Snapper timers where default. |
| **D‑09** | Replication units | Timers + udev‑triggered service for USB catch‑up. |
| **D‑10** | Udev rule | Fires replication service when backup disk appears. |
| **D‑11** | Scrub & balance timers | Monthly scrub, quarterly balance with alerting. |
| **D‑12** | Health‑alert service | Warn if backup stale or scrub fails. |
| **D‑13** | LUKS header backup | Offline copy of encrypted backup‑partition header + hash. |
| **D‑14** | Swing‑space reclaim script | One‑liner to grow NTFS or Btrfs when cushion deleted. |
| **D‑15** | Tool‑set documentation | Lists chosen utilities, reasons, roles, usage policy. |
| **D‑16** | Trip‑Checklist | Manual pre‑trip snapshot & cloud push instructions. |
| **D‑17** | Quarterly‑Maintenance checklist | Tasks incl. free‑space check & quota enabling. |
| **D‑18** | Procurement list | USB disks, labels, antistatic pouches for future buys. |
| **D‑19** | Multi‑OS backup guide | How Linux, Windows & Android feed the same encrypted repo. |
| **D‑20** | Restore capabilities summary | Matrix + cheat‑sheets for file & bare‑metal restore from every tier. |
| **D‑21** | Android backup guide | Termux/Restic, SeedVault, TWRP paths; schedule & restore steps. |
| **D‑22** | Off‑site backup guide & cost‑update script | Provider comparison plus `update_costs.py` that refreshes cost table quarterly. |
| **D‑23** | Version‑skew guide | Policies to avoid config corruption when multiple roots share `/home`. |
| **D‑24** | Distro recommendations & Secure‑Boot notes | Ranked list (Tumbleweed etc.), SB workflow cheatsheet. |
| **D‑25** | Implementation plan | Phase tasks mapped to milestones; live progress checklist. |
