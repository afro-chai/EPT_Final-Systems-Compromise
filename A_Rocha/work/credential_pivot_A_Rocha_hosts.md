# Credential pivot plan — A_Rocha hosts (`.100` `.101` `.102`)

**Secrets live only in** [`credentials_handoff_SECRETS.md`](credentials_handoff_SECRETS.md) **(gitignored).** This file is safe to commit: it names **vectors**, not passwords.

## How the deck maps to lateral movement

From **Final Presentation.pdf** (team narrative):

| Teammate lane | Idea you can reuse on your IPs |
|---------------|----------------------------------|
| **GANYMEDE (.125)** | **Product ≠ banner** (WeOnlyDo vs FreeSSHd) → always fingerprint **service**, not substring of the banner. |
| **GANYMEDE post-ex** | **SYSTEM** on a **JUPITER** member → **LSA / SAM / MSCache** artifacts → **password reuse** and **hash** cracking against **other members** (e.g. **CALLISTO**). |
| **METIS / THEBE** | **Domain controllers** expose **DefaultPassword**-style material and **NTLM** — useful for **crack / spray** against **workstation SMB & RDP**. |
| **XENA (.111)** | **Tomcat / Bitnami manager** creds → try **HTTP Basic** / **`/manager/html`** on **any** host **:8080** in the lab (your **.102** stack is **LAMPP**, not Bitnami—still worth a **30-second curl**). |

## Prioritized attempts against **your** three IPs

### `10.20.160.101` (CALLISTO — Windows 7, **JUPITER**)

1. **RDP 3389** — use **GANYMEDE**-recovered **local Administrator** cleartext from the secrets file with username **`Administrator`** against **CALLISTO** (labs often reuse recovered passwords on sibling **JUPITER** hosts).  
2. **SMB 445** — same pair with **CrackMapExec** (`smb`, `winrm` if open): `crackmapexec smb 10.20.160.101 -u Administrator -p '<from secrets>'`  
3. **Spray METIS/THEBE `DefaultPassword`** strings (see secrets file) with **`Administrator`** / **`administrator`** over **SMB** and **RDP**.  
4. **Pass-the-hash** — **METIS/THEBE `Administrator` NTLM`** (32-hex in secrets file) with **impacket** / **MSF** **psexec** *only if* course ROE allows **PTH** and AV is not a graded constraint.  
5. **MySQL 3306** — try **same clear passwords** as **app** users (weak lab pattern); expect **1042** until server-side DNS is fixed—then **`phpMyAdmin`** on **80** with those creds.

### `10.20.160.100` (Windows XP)

1. **SMB** — **null** session and **legacy** modules are the historical XP story; add **spray** of **one** cleartext you trust from the handoff (short attempts only).  
2. **RDP** — if **3389** is open, same **Administrator** / **DefaultPassword** tries as **.101** (XP may use **local** accounts only—adjust usernames per your **enum**).

### `10.20.160.102` (Linux / dotProject — **not** domain-joined)

1. **Tomcat / manager** — quick **`curl -sI http://10.20.160.102:8080/manager/html`** (and **8443**) with **XENA**-style creds from the secrets file—usually **negative** on LAMPP, but fast to rule out.  
2. **MySQL** — if **FTP** or **RFI** ever surfaces a **`my.cnf` / app config`**, use that user/pass **locally** against **3306** on **.101** first (MySQL more likely there); on **.102** only if **mysqld** listens on **0.0.0.0**.  
3. **SSH** — only if **nmap** shows **22** and the team shares a **Linux** password (not in your current paste—ask **T_Amor** for **.112** / **.123** style creds if they found reuse).

## Privilege escalation (after any shell on `.101` / `.100`)

- **Local:** **token** / **unquoted service path** / **always install elevated** — classic Win7 checklist.  
- **Domain:** if **`JUPITER\user`** shell, **`net group "Domain Admins" /domain`**, **bloodhound**-lite **manual** (who owns **METIS/THEBE**), **Kerberoast** only if **SPNs** exist and ROE allows.

## Hygiene

- Re-type secrets from **`credentials_handoff_SECRETS.md`** into commands **locally**; do not paste into **Slack** / **GitHub issues**.  
- When a spray works, **screenshot** + **`101-NNN` / `100-NNN`** naming, then summarize **here** without repeating the password in committed markdown.
