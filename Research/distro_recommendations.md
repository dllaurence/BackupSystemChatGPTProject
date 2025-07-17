# Rolling Linux Distro Recommendations (NVIDIA & Secure Boot Friendly)

## Selection Criteria
- Rolling or near‑rolling cadence for up‑to‑date hardware enablement  
- Proven proprietary NVIDIA driver support without manual DKMS drama  
- Secure Boot (SB) works out‑of‑the‑box or with one guided MOK enrolment  
- Large package repositories or easy access to community packaging  
- Minimal day‑to‑day maintenance

## Short‑List (ranked)
| Rank | Distro / Spin | Secure Boot Status | NVIDIA Driver Path | Hassle Index (1 = minimal) | Notes |
|------|---------------|--------------------|--------------------|----------------------------|-------|
| 1 | **openSUSE Tumbleweed / Aeon** | 🟢 Green – shim + YaST guided MOK | Repo‑built signed modules | **1** | Reference “stable‑rolling” experience |
| 2 | **Nobara Project** (Fedora remix) | 🟢 Green – inherits Fedora shim, pre‑installed driver | Auto‑signed akmods | **1‑2** | Ideal gaming test root |
| 3 | **Siduction** (Debian Sid) | 🟢 Green – Debian shim + DKMS auto‑sign | `nvidia-driver` DKMS | **2** | Pure rolling; snapshots mitigate risks |
| 4 | **Pop!_OS** (NVIDIA ISO) | 🟡 Yellow – System76 shim, helper script for re‑sign | Pre‑installed driver | **2‑3** | Best Optimus UX; Ubuntu base |
| 5 | **Manjaro** (stable) | 🟠 Orange – community shim, manual key | Pre‑built nvidia‑open/dkms; manual sign | **3** | AUR depth vs SB friction |
| 6 | **NixOS unstable** | 🟠 Orange – sbctl module, manual first run | Declarative rebuild + user hook | **4** | For experimentation only |

*Dropped:* Solus, Void, pure Debian Testing – weaker SB or NVIDIA stories.

## Recommended Partition Strategy
| Root Slot | Purpose | Suggested Distro |
|-----------|---------|------------------|
| **Work (primary)** | Daily driver, lowest maintenance | **openSUSE Tumbleweed** (or **Aeon**) |
| **Test #1** | Gaming‑centric experimentation | **Nobara Project** |
| **Test #2** | Rolling Debian experience | **Siduction** |
| **Optional Tests** | Optimus convenience or declarative tinkering | **Pop!_OS**, **Manjaro**, **NixOS** |

## Secure Boot Workflow Cheatsheet
| Task | openSUSE / Nobara | Siduction | Manjaro | NixOS |
|------|------------------|-----------|---------|-------|
| First enrolment | YaST / GNOME Software MOK dialog | `mokutil --import key.der` | Manual shim + key | `sbctl enroll-keys` |
| Kernel updates | Already signed | Already signed | Already signed | sbctl hook |
| NVIDIA updates | Auto‑signed repo module | Auto‑signed akmods | `dkms install && sign-file` | User hook |

## TL;DR
Stay on **Tumbleweed/Aeon** for the main root, pair it with **Nobara** (and optionally **Siduction**) for your experimental partition. You gain top‑tier NVIDIA performance *and* a Secure Boot process that won’t derail your workflow.

