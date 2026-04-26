# PWN2 — work log, dead ends, and archived attempts

Purpose: this file holds **what was tried and did not yield results**, raw notes, and reusable command blocks.  
The root [`README.md`](../README.md) is now the clean execution plan.

---

## Session continuity (extension + restart)

- Session restart required after extension.
- Recreate local folders each fresh session:

```bash
mkdir -p ~/logs ~/scans ~/loot
echo afrocha; date
```

---

## Tried but no yield (moved from root README)

### A) `10.20.160.63` (`MATHIS`) SMB lane

| Attempt | Result |
|---------|--------|
| `ms17_010_eternalblue` check | Not vulnerable |
| `ms17_010_psexec` check | Not vulnerable |
| `auxiliary/scanner/smb/smb_ms17_010` | Not vulnerable on `.63` |
| `enum4linux -a` | Mostly `ACCESS_DENIED` |
| `rpcclient` null session | `NT_STATUS_ACCESS_DENIED` |
| `smbclient -L -N` | No useful shares |
| `smb_login` starter wordlists | All combinations failed |

Key evidence: `04_`, `05_`, `08_`, `09_`, `10_`, `27_`–`37_`, `46_`.

### B) `10.20.160.112` (`JUAN`) proof search pain points

| Attempt | Result |
|---------|--------|
| `dir C:\Users\ /s /b *proof*.txt` | No immediate hits |
| `dir C:\Users\ /s /b *local*.txt` | No immediate hits |
| Whole-drive wildcard search | Very noisy scrollback |
| One-liner redirected to wrong output file | Misnamed output (`local_hits.txt`) |
| Path typo (`Administator`) | False negative (`File Not Found`) |

Key evidence: `24_`, `38_`, `39_`, `40_`, `41_`, `42_`, `43_`.

---

## Archive: old “Part 3.5 alternate proof retrieval”

This section was removed from root README and parked here for reference.

### FTP / traversal / file-read methodology (only if service exists)

Use only when scan shows a matching service (ex: FTP on `21/tcp`):

```bash
echo afrocha; date
nmap -sn 10.20.160.10-200 -oA ~/scans/discovery_refresh
nmap -p 21,80,8080,443,139,445 --open -sV 10.20.160.63 10.20.160.112
```

Current observed state from prior run:

- `.63`: `139/445` only
- `.112`: `8080` only
- No FTP on either host in that scan

If FTP appears later:

```text
search ftp
search type:auxiliary platform:windows ftp
use auxiliary/<matching_module>
set RHOSTS <target_ip>
set RPORT 21
set PATH Users/Juan/Desktop/proof.txt
run
```

Then iterate likely paths and validate with stamped evidence.

---

## Reusable command blocks

### `.63` quick SMB recon loop

```bash
echo afrocha; date
nmap -p139,445 -sV -sC 10.20.160.63 -oA ~/scans/host63_smb_default
nmap -p139,445 --script smb-vuln-ms17-010 10.20.160.63 -oA ~/scans/host63_ms17_script
```

```text
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.20.160.63
set RPORT 445
check
```

### `.63` credential lane (if permitted)

```bash
printf '%s\n' administrator guest juan > ~/loot/users_63.txt
printf '%s\n' administrator password pwn2 juan Welcome1 Spring2026! > ~/loot/pass_63.txt
```

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

### `.112` proof search helper

```cmd
echo afrocha %date% %time%
dir C:\ /s /b proof.txt
dir C:\ /s /b local.txt
for /r C:\ %i in (proof.txt) do @echo %i>> "%USERPROFILE%\Desktop\proof_exact.txt"
for /r C:\ %i in (local.txt) do @echo %i>> "%USERPROFILE%\Desktop\local_exact.txt"
```

---

## Screenshot index (reference)

All evidence images are in [`../1_Screenshots/`](../1_Screenshots/).

- Discovery/host ID: `01_`, `02_`
- SMB attempts on `.63`: `03_` to `10_`, `27_` to `37_`, `44_` to `46_`
- Exploit success on `.112`: `14_`, `15_`
- Proof-search attempts/lessons: `24_`, `38_` to `43_`
