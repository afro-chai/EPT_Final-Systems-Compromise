# Work (scratch)

Session transcripts, command snippets, loot exports, and rough notes. Keep **passwords and live tokens out of git**—paste redacted copies here if you need a paper trail.

**Team cred pivot (`.100` / `.101` / `.102`):** [`credential_pivot_A_Rocha_hosts.md`](credential_pivot_A_Rocha_hosts.md). **Full teammate dump (gitignored):** [`credentials_handoff_SECRETS.md`](credentials_handoff_SECRETS.md) — copy from teammate / sheet; **never** `git add` that file (listed in repo [`.gitignore`](../../.gitignore)).

---

## Per-host work (scoped IPs)

| IP | System | Plan, commands, evidence |
|----|--------|-------------------------|
| **`10.20.160.102`** | Linux / dotProject | [`README.md`](10.20.160.102/README.md) · [`Screenshots/`](10.20.160.102/Screenshots/) · [`credential_pivot.md`](10.20.160.102/credential_pivot.md) · [`unfruitful_attempts.md`](10.20.160.102/unfruitful_attempts.md) |
| **`10.20.160.101`** | Windows 7 (CALLISTO) | [`README.md`](10.20.160.101/README.md) · [`Screenshots/`](10.20.160.101/Screenshots/) · [`credential_pivot.md`](10.20.160.101/credential_pivot.md) · [`unfruitful_attempts.md`](10.20.160.101/unfruitful_attempts.md) |
| **`10.20.160.100`** | Windows XP (ADRASTEA) | [`README.md`](10.20.160.100/README.md) · [`Screenshots/`](10.20.160.100/Screenshots/) · [`credential_pivot.md`](10.20.160.100/credential_pivot.md) · [`unfruitful_attempts.md`](10.20.160.100/unfruitful_attempts.md) |

**Dead-end lanes:** [`unfruitful_attempts/README.md`](unfruitful_attempts/README.md) (index) — per-host copies under **`unfruitful_attempts.md`** in each IP folder.

**Nessus / host specs:** [A_Rocha `README.md`](../README.md#nessus--my-hosts-specs--findings).

---

## `10.20.160.124` — Domain Controller HIMALIA (`Jupiter.local`)

**Ownership / ROE:** This host is **teammate J_Solis** scope on the map; only probe with **written OK** and **instructor ROE**. These captures are **lightweight fingerprint** (anonymous LDAP + SMB/Kerberos scripts), not an exploit chain.

**What the evidence shows:** **`ldapsearch -x -H ldap://10.20.160.124 -s base namingcontexts`** returns **`DC=Jupiter,DC=local`** (and partition naming contexts) — confirms **AD** on **`.124`**. **`nmap -Pn -p88,445 --script krb5-enum-users,smb-os-discovery,smb-security-mode`** shows **Kerberos (`88`)** and **SMB (`445`)**, **`smb-os-discovery`** → **Windows Server 2008 R2 SP1**, computer **HIMALIA**, domain/forest **`Jupiter.local`**; **`smb-security-mode`** → **`message_signing: required`** (typical for a **DC**).

### Evidence (`124-NNN_*` — for this host octet)

| # | File | What it shows |
|---|------|----------------|
| **124-001** | [`124-001_ldapsearch_namingcontexts_jupiter_local_dc_tm6_afrocha.png`](../Screenshots/124-001_ldapsearch_namingcontexts_jupiter_local_dc_tm6_afrocha.png) | **`ldapsearch`** anonymous **`-s base namingcontexts`** on **`.124`** → **`Jupiter.local`** naming contexts |
| **124-002** | [`124-002_nmap_krb_smb_himalia_dc_tm6_afrocha.png`](../Screenshots/124-002_nmap_krb_smb_himalia_dc_tm6_afrocha.png) | **`nmap`** **`88,445`** + **`krb5-enum-users` / `smb-os-discovery` / `smb-security-mode`** — **HIMALIA**, **Win 2008 R2**, **`message_signing: required`**, stamp **`TM6_afrocha`** |

---

## Ordering suggestion (one operator on KALI6)

1. **.102** — **dotProject RFI / web** first (validated path); **ZAP** flags **`login`** parameter tampering → time-box **SQLi / auth** follow-up; Shellshock and generic MSF noise archived under [`unfruitful_attempts/`](unfruitful_attempts/README.md).  
2. **.100** — **XP** is still high value, but **run `nmap` on `139,445,3389` before `enum4linux`**: if **null SMB** stays dead, use **creds / RDP / vuln `check`** rather than repeating anonymous enum. After **shell**, **`ping`** **`.101` / `.102`** confirms **L3** reach for **pivot** — see [`10.20.160.100/README.md`](10.20.160.100/README.md) (**`ping`** / **`autoroute`**).  
3. **.101** — **Win7** harder target; use creds or results from **.100** lateral moves if the scenario allows.

Document **what you ran**, **what failed** ([`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)), and **screenshots** under each host’s **`Screenshots/`** folder (and shared [`../Screenshots/`](../Screenshots/) for STEPfwd, **`.124`**, Nessus) before promoting to team [`../../1_Screenshots/`](../../1_Screenshots/).
