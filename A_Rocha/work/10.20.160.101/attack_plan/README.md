# Attack plan — `10.20.160.101` (CALLISTO / Windows 7)

**Host:** **`10.20.160.101`** · **NetBIOS:** **CALLISTO** · **Workgroup:** **JUPITER**.

**Navigation:** [Host README (full runbook)](../README.md) · [Screenshots](../Screenshots/) · [Credential pivot](../credential_pivot.md) · [Unfruitful log](../unfruitful_attempts/README.md) · [Work index](../../README.md)

---

## Phase 0 — Preconditions

- Export **`RHOST=10.20.160.101`**; confirm **third octet** is **`.160.*`** (not **`.150.*`**) before any **MSF** **`RHOSTS`**.
- **EternalBlue** uses **`RPORT 445`** only — never **80**.

## Phase 1 — Discovery

- **`nmap --top-ports 1000`** (or focused service scan): **21 FTP**, **80/443 HTTP**, **135/139/445 SMB**, **3306 MySQL**, **3389 RDP**, dynamic **RPC**.
- **Null SMB:** **`smbclient`**, **CME**, **`enum4linux`** — expect **denied** or **no share list** on this image; treat as signal to move on (see [unfruitful log](../unfruitful_attempts/README.md)).

## Phase 2 — Low-friction access (validated in this lab)

- **Anonymous FTP** (`anonymous` / email-style password) → list **`/forbidden`**, **`get`** **`.htaccess` / `.htpasswd`** if ROE allows.
- **HTTP** Basic on **`/forbidden/`** using **`.htpasswd`** material → note **XAMPP “local network only”** **403** (needs host access or another misconfig to bypass).

## Phase 3 — SMB exploit lane (likely dead on this VM)

- **`nmap --script smb-vuln-ms17-010`** and **`ms17_010_eternalblue`** **`check`** on **445** — if **not vulnerable**, stop forcing unless instructor allows (see [unfruitful log](../unfruitful_attempts/README.md) for **`ForceExploit`** / **`0xc000018d`** captures).

## Phase 4 — Credentialed pivot

- Reuse passwords / hashes from **`.100` SAM**, teammate handoff, or sheet — **RDP**, **SMB**, **MySQL**, **WinRM** per [`../credential_pivot.md`](../credential_pivot.md).

## Phase 5 — Reporting

- Evidence **`101-NNN_*`** under [`../Screenshots/`](../Screenshots/); document **EternalBlue** negatives and **FTP/HTTP** positives for the report.

---

**Full command blocks and evidence tables:** [../README.md](../README.md).
