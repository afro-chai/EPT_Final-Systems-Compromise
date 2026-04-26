# Attack plan - `10.20.160.124` (HIMALIA / AD DC)

**Navigation:** [Host README](../README.md) · [Screenshots](../Screenshots/) · [Credential pivot](../credential_pivot.md) · [Unfruitful log](../unfruitful_attempts/README.md) · [Work index](../../README.md)

## Phase 1 - Baseline recon
- `ldapsearch -x -H ldap://10.20.160.124 -s base namingcontexts`
- `nmap -Pn --top-ports 1000 -sV 10.20.160.124`
- `nmap -Pn -p88,389,445,3389 --script smb-os-discovery,smb-security-mode 10.20.160.124`

## Phase 2 - BlueKeep hypothesis (safe-first)
- Module context capture:
  - `search type:exploit platform:linux cgi_bash` is not relevant here; use RDP module context from screenshots.
  - `use exploit/windows/rdp/cve_2019_0708_bluekeep_rce`
  - `show info`
  - `show options`
- Set only discovery-safe options first and run checks where available; do not force crash-prone runs on a DC without explicit instructor ROE.

## Phase 3 - Credential and protocol validation
- Prioritize credentialed SMB/RDP/WinRM validation from approved handoff before risky kernel exploit paths.
- Record exact command, target response, and whether result is positive, negative, or inconclusive.

## Phase 4 - Reporting
- Keep evidence series in `Screenshots/`:
  - `124-003` (`nmap` top-ports fingerprint)
  - `124-004` (BlueKeep `show info` targets/support)
  - `124-005` (BlueKeep options / `RHOSTS` / `RPORT`)
- Move any failed lane details to `../unfruitful_attempts/README.md` with embedded screenshots.
