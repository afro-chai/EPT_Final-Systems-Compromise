# Unfruitful attempts — `10.20.160.102`

Archived lanes and mistakes from the **plan of attack** phase. The **current** `.102` workflow lives in [`../README.md`](../README.md) (section **`10.20.160.102`**). Evidence images stay in [`../../Screenshots/`](../../Screenshots/).

---

## Shellshock / CGI / Metasploit (no shell via this lane)

**Hypothesis:** Nessus and stack banners suggested **Shellshock** via **`/cgi-bin/`**; **Nikto** also flagged **`/cgi-bin/printenv`** and **`/cgi-bin/test-cgi`**.

| Attempt | What we ran | Outcome |
|---------|-------------|---------|
| Bare **`/cgi-bin`** | `curl` without a script name | **404** — not proof that no CGI exists |
| **Nmap `http-shellshock`** | `--script-args http-shellshock.uri=/cgi-bin/test.cgi` | **No** `http-shellshock:` finding — stock path likely absent |
| **Nikto** | `nikto -h "http://$RHOST_102"` | Reported **printenv** / **test-cgi** as *appearing* Shellshock-vulnerable |
| **Nmap** (follow-up) | `http-shellshock.uri=/cgi-bin/printenv` or **`/cgi-bin/test-cgi`** | Worth a retry if revisiting; primary effort moved to **dotProject** |
| **Metasploit** `exploit/multi/http/apache_mod_cgi_bash_env_exec` | `TARGETURI /cgi-bin/printenv`, meterpreter payload | **`check`** → **The target is not exploitable** — see [`exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png`](../../Screenshots/exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png) |

**`RPATH` mistake (once):** In that module, **`RPATH`** is the **CmdStager filesystem prefix** (default **`/bin`**), **not** the CGI URL. Only **`TARGETURI`** holds **`/cgi-bin/...`**.

**Optional retries (low priority):** `TARGETURI /cgi-bin/test-cgi`, `set CVE CVE-2014-6278`, alternate **`HEADER`**. If still negative, treat **Shellshock** as **closed** for this host.

**Recon screenshots (context):** [`recon_curl_cgi-bin_404_headers_tm6_afrocha.png`](../../Screenshots/recon_curl_cgi-bin_404_headers_tm6_afrocha.png), [`recon_nmap_http_shellshock_test_cgi_tm6_afrocha.png`](../../Screenshots/recon_nmap_http_shellshock_test_cgi_tm6_afrocha.png), Nikto [`recon_nikto_102_http80_banner_redirect_tm6_afrocha.png`](../../Screenshots/recon_nikto_102_http80_banner_redirect_tm6_afrocha.png) / [`recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png`](../../Screenshots/recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png).

---

## Metasploit — no dotProject module; wrong searches

| Attempt | Detail |
|---------|--------|
| **`search dot`** | Matches **DotNetNuke**, **dotnet**, **Oracle**, **Struts** — **not** **dotProject**. See [`102-002_msf_search_dot_noise_tm6_afrocha.png`](../../Screenshots/102-002_msf_search_dot_noise_tm6_afrocha.png). |
| **`exploit/linux/http/php_imap_open_rce`** | Targets **PrestaShop / SuiteCRM / e107 / Horde** (+ **custom**), **not** dotProject — no stock **`TARGETURI /dotproject/`**. |

---

## `searchsploit` / generic PHP–Apache rows (deprioritized on this host)

Many **`searchsploit apache 2.2.21`** / **`php 5.3.8`** hits are **other products** (Drupal, WordPress plugins, etc.). Generic **PHP-CGI** (**CVE-2012-1823** class) and **`php-cgi`** routing did not drive the successful path — this site behaved as **mod_php** under Apache. **OPTIONS** memory leak / **DoS** modules are report-only for most labs.

Screenshots: [`searchsploit_apache_2_2_21_tm6_afrocha.png`](../../Screenshots/searchsploit_apache_2_2_21_tm6_afrocha.png), [`searchsploit_php_5_3_8_tm6_afrocha.png`](../../Screenshots/searchsploit_php_5_3_8_tm6_afrocha.png), [`searchsploit_dotproject_tm6_afrocha.png`](../../Screenshots/searchsploit_dotproject_tm6_afrocha.png).

---

## RFI callback mistakes (before success)

| Issue | Fix |
|-------|-----|
| **`Connection refused`** from **`.102`** to Kali | Wrong **`LHOST`**, server not listening, **`--bind 0.0.0.0`**, or firewall — see [`102-003_rfi_gantt_include_connection_refused_tm6_afrocha.png`](../../Screenshots/102-003_rfi_gantt_include_connection_refused_tm6_afrocha.png). |
| **`proof.txt` 404** on Kali | File was under **`~/loot/`** while **`http.server`** ran from **`~/rfi_poc/`** — serve files from **the same directory** as the listener. |
| **`curl https://…:8080`** to Python **`http.server`** | Use **`http://`** only; **`https://`** causes **400 Bad request version** (TLS bytes to plain HTTP). |

---

## Original “Shellshock-first” plan text (superseded)

Early notes prioritized **web → CGI → Shellshock** parallel to **dotProject**. Enumeration (`curl` **`/cgi-bin/`**, **`nmap http-shellshock`**, **`gobuster`** on **`/cgi-bin/`**) remains valid **background** work; **exploitation** effort shifted after **dotProject 2.1.6** fingerprint and **`gantt.php`** RFI confirmation.

---

## COA 2 — generic `searchsploit` rows (not the winning path on `.102`)

For reference, **`searchsploit apache 2.2.21`**, **`php 5.3.8`**, and broad PHP **CGI** / **OPTIONS** / **DoS** rows were catalogued. Many hits target **other CMSs** or need **`php-cgi`** layout; the **successful** lane was **dotProject-specific RFI** (`gantt.php`), not **CVE-2012-1823**-style CGI on this host.
