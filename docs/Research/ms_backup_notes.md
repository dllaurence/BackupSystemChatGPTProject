# Microsoft Built‑In Backup Options — Why We Aren’t Using Them as Primary

| Component | Scope & Convenience | Limitations (Deal‑Breakers) | Verdict |
|-----------|--------------------|-----------------------------|---------|
| **Windows Backup app** (Win 11 23H2+) | One‑click cloud backup of Desktop, Documents, Pictures, Music, Videos + some settings; auto‑runs. | Backs up only to **OneDrive**; counts against 5 GB / 1 TB M365 quota; **no bare‑metal image** — requires clean reinstall before restore; data encrypted only with Microsoft‑held keys. | **Supplementary at best**; fails “bare‑metal” & “self‑owned keys” must‑haves. |
| **File History** | Continuous local versioning to USB/NAS; easy per‑file restore. | No cloud target; rumored future deprecation; doesn’t capture full system. | Keep as an *extra* on Windows PCs but not part of off‑site plan. |
| **System‑Image (“Windows 7 Backup”)** | Block‑level VHDX image to USB/share. | Deprecated UI; no cloud target; VHD restores only via legacy toolchain. | Superseded by **Veeam Agent Free** in our plan. |
| **OneDrive folder sync / Personal Vault** | Transparent folder redirection; ransomware detection; MFA vault. | Same privacy model as Backup app (Microsoft holds keys); limited to known folders; still no OS image. | Good for quick self‑service restores but doesn’t meet full requirements. |

## Key Reasons for Excluding Microsoft Cloud Backup as Primary
1. **Privacy / Encryption** – Data is encrypted in transit and at rest, but **Microsoft controls the keys**; we require end‑to‑end encryption with keys we own (Restic/Kopia).  
2. **Bare‑Metal Restore** – None of the built‑ins can re‑image a blank drive without first reinstalling Windows; Veeam Agent Free provides full‑disk recovery that integrates with our Restic repo.  
3. **Cross‑Platform Consistency** – Built‑ins serve Windows only; our household plan unifies Linux, Windows, and Android in one encrypted repository.  
4. **Cost Predictability** – OneDrive storage is tied to Microsoft 365; scaling beyond 1 TB per user is costly compared to Backblaze/Hetzner.  
5. **Retention Flexibility** – Restic/Kopia offer programmable prune policies; OneDrive holds 30‑day version history max.

### When It’s Still Useful
* Fast restore of small user files on the same PC.  
* Hands‑off safety net for non‑technical family members.  
* Personal Vault for a handful of sensitive docs.

> **Conclusion:** Microsoft’s solutions remain enabled for *convenience*, but the official off‑site backup of record is our Restic/Kopia repository (encrypted with self‑owned keys) plus Veeam images for bare‑metal restores.

