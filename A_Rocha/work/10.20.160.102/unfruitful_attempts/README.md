# Unfruitful attempts — `10.20.160.102` (Linux / dotProject)

Lanes that **did not** yield a shell (or were **operator mistakes** / **false-negative `check`** noise) alongside **validated** wins (**RFI**, **Shellshock Meterpreter** — see section 1 update and [host README](../README.md)). Screenshots are **embedded** below.

**Navigation:** [Host README](../README.md) · [Attack plan](../attack_plan/README.md) · [Screenshots folder](../Screenshots/) · [Work index](../../README.md)

---

## 1 — Shellshock / CGI / Metasploit (no shell via this lane)

**Hypothesis:** Nessus and banners suggested **Shellshock** via **`/cgi-bin/`**; **Nikto** flagged **`printenv`** / **`test-cgi`**.

| Attempt | What we ran | Outcome |
|---------|-------------|---------|
| Bare **`/cgi-bin`** | `curl` without script | **404** |
| **Nmap `http-shellshock`** | `http-shellshock.uri=/cgi-bin/test.cgi` | No **`http-shellshock:`** finding |
| **Nikto** | `nikto -h "http://$RHOST_102"` | Flags *appearing* Shellshock paths |
| **Metasploit** `apache_mod_cgi_bash_env_exec` | **`TARGETURI /cgi-bin/printenv`** | **`check`** → **not exploitable** |

**Note:** **`RPATH`** in that module is **CmdStager prefix** (`/bin`), **not** the CGI URL — only **`TARGETURI`** holds **`/cgi-bin/...`**.

**Update (2026-04-25):** A later **`msfconsole`** session used the **same module** with **`set targeturi /cgi-bin/test-cgi`** and **`run`** (reverse handler **`10.20.150.106:4444`**) — **Meterpreter** on **`.102`** (**session 5** in report capture), **`sysinfo`** **CentOS 6.3**, **`getuid`** **`no-user @ Elara`**, **`shell`** + **`echo afrocha; date`**. The gap vs the **`printenv`** row above is **`check`** on **`printenv`** vs **`run`** on **`/cgi-bin/test-cgi`**.

### Evidence (embedded)

![102-011 — MSF search, set targeturi test-cgi, run](../Screenshots/102-011_msf_search_apache_mod_cgi_bash_env_set_targeturi_test_cgi_run_tm6_afrocha.png)

![102-012 — Meterpreter session 5 sysinfo getuid shell stamp](../Screenshots/102-012_msf_shellshock_meterpreter_session5_sysinfo_getuid_shell_afrocha_tm6_afrocha.png)

![102-009 — Shellshock apache_mod_cgi Meterpreter sysinfo shell Elara](../Screenshots/102-009_msf_shellshock_apache_mod_cgi_meterpreter_sysinfo_shell_elara_tm6_afrocha.png)

![exploit_msf_shellshock_printenv — check not vulnerable](../Screenshots/exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png)

![recon_curl_cgi-bin — 404 headers](../Screenshots/recon_curl_cgi-bin_404_headers_tm6_afrocha.png)

![recon_nmap_http_shellshock_test_cgi](../Screenshots/recon_nmap_http_shellshock_test_cgi_tm6_afrocha.png)

![recon_nikto_102_http80_banner_redirect](../Screenshots/recon_nikto_102_http80_banner_redirect_tm6_afrocha.png)

![recon_nikto_102_dotproject_paths_shellshock](../Screenshots/recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png)

---

## 2 — Metasploit — no dotProject module; wrong `search dot`

| Attempt | Detail |
|---------|--------|
| **`search dot`** | Hits **DotNetNuke**, **dotnet**, etc. — **not** **dotProject**. |
| **`php_imap_open_rce`** | Wrong products; no stock **`/dotproject/`** URI. |

![102-002 — msf search dot noise](../Screenshots/102-002_msf_search_dot_noise_tm6_afrocha.png)

---

## 3 — `searchsploit` / generic PHP–Apache rows (not the winning path)

Many **Apache 2.2.21** / **PHP 5.3.8** rows target **other CMSs** or **php-cgi** layout; this host behaved as **mod_php**.

![searchsploit_apache_2_2_21](../Screenshots/searchsploit_apache_2_2_21_tm6_afrocha.png)

![searchsploit_php_5_3_8](../Screenshots/searchsploit_php_5_3_8_tm6_afrocha.png)

![searchsploit_dotproject menu](../Screenshots/searchsploit_dotproject_tm6_afrocha.png)

---

## 4 — RFI callback mistakes (before success)

| Issue | Fix |
|-------|-----|
| **Connection refused** from **`.102`** to Kali | Wrong **`LHOST`**, listener not **`0.0.0.0`**, or firewall. |
| **`proof.txt` 404** on Kali | **`http.server`** cwd ≠ file location. |
| **`curl https://…:8080`** to Python server | Use **`http://`** only. |

![102-003 — RFI gantt include, connection refused](../Screenshots/102-003_rfi_gantt_include_connection_refused_tm6_afrocha.png)

---

## 5 — Plan evolution (RFI first, then Shellshock **run**)

Enumeration of **`/cgi-bin/`** stayed relevant: **RFI** (**`102-004`**) proved file read/inclusion first; **Shellshock** via **`apache_mod_cgi_bash_env_exec`** later delivered **Meterpreter** (**`102-009`**) once **`TARGETURI`** matched a real **bash-backed CGI** (see section 1 update). Both are **validated** for the report — see [host README](../README.md).

---

## 6 — COA 2 — generic `searchsploit` catalogue (reference only)

**Primary coded lanes on this host:** **dotProject RFI** (**EDB-22708**) **and** **Shellshock** (**`apache_mod_cgi_bash_env_exec`**). Generic **CVE-2012-1823**-style **PHP-CGI** rows were **not** the match for this stack.

---

**Central index (all hosts):** [`../../unfruitful_attempts/README.md`](../../unfruitful_attempts/README.md)
