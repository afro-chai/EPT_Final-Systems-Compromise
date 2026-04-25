# Credential pivot plan — A_Rocha hosts (`.100` `.101` `.102`)

**Secrets live only in** [`credentials_handoff_SECRETS.md`](credentials_handoff_SECRETS.md) **(gitignored).** This file is safe to commit: use **placeholders** below; paste real passwords / hashes **only in your shell**, never into git.

**Per-host command cheat sheets (split by system):**

| IP | File |
|----|------|
| **`10.20.160.101`** | [`10.20.160.101/credential_pivot.md`](10.20.160.101/credential_pivot.md) |
| **`10.20.160.100`** | [`10.20.160.100/credential_pivot.md`](10.20.160.100/credential_pivot.md) |
| **`10.20.160.102`** | [`10.20.160.102/credential_pivot.md`](10.20.160.102/credential_pivot.md) |

---

## 0 — Prep (Kali, one terminal)

```bash
echo TM6_afrocha; date
ip -br a    # confirm lab IP for any callback / RDP source if needed

# IPs (short names)
export H101=10.20.160.101
export H100=10.20.160.100
export H102=10.20.160.102

# Open credentials_handoff_SECRETS.md in a second window — copy values as you go.
# Optional: export secrets ONLY for this shell session (still do not log to git):
#   export TRY_PASS='...'     # e.g. GANYMEDE recovered Administrator cleartext
#   export METIS_NT='...'     # 32-hex NTLM for METIS\Administrator (no "NTLM:" prefix)
```

---

## 4 — After a **Windows** shell (`.101` / `.100`) — priv esc checklist

```text
whoami /groups
systeminfo
net user
net localgroup administrators
# Unquoted paths / weak service perms / token abuse — per rubric and ROE
```

If you are **`JUPITER\`** user:

```text
net group "domain admins" /domain
net group "enterprise admins" /domain
```

---

## 5 — Optional: crack **NTLM** offline (METIS/THEBE **Administrator**)

Only on **lab** hashes you are authorized to attack.

```bash
echo '<NTLM_HASH>' > ~/hashes/metis_admin.ntlm
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ~/hashes/metis_admin.ntlm
# or hashcat -m 1000 ...
```

If **John** cracks it, return to **[`10.20.160.101/credential_pivot.md`](10.20.160.101/credential_pivot.md)** **RDP / SMB** steps with the **cleartext**.

---

## How the deck maps to lateral movement

| Teammate lane | Idea you can reuse on your IPs |
|---------------|----------------------------------|
| **GANYMEDE (.125)** | **Product ≠ banner** (WeOnlyDo vs FreeSSHd) → fingerprint **actual product**, not a substring of the SSH banner. |
| **GANYMEDE post-ex** | **SYSTEM** on **JUPITER** member → **LSA / SAM / MSCache** → **reuse** cleartext + **crack** hashes → **CALLISTO (.101)** **RDP/SMB**. |
| **METIS / THEBE** | **DC `DefaultPassword`** + **`Administrator` NTLM`** → **spray** / **PTH** onto **workstations**. |
| **XENA (.111)** | **Tomcat / Bitnami** → quick **curl** to **`.102:8080`** **manager** (likely negative on **LAMPP**). |

---

## Hygiene

- Re-type secrets from **`credentials_handoff_SECRETS.md`** into commands **locally**; do not paste into **Slack** / **GitHub issues**.  
- When a spray works, **screenshot** + **`101-NNN` / `100-NNN`** naming; update the relevant per-host doc with **“works: method + user”** only — **not** the password.
