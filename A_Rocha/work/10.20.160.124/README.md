**Navigation:** [Work index](../README.md) ù [Attack plan](./attack_plan/README.md) ù [Credential pivot (hub)](../credential_pivot_A_Rocha_hosts.md) ù [Creds ù this host](./credential_pivot.md) ù [Dead ends](./unfruitful_attempts/README.md) ù [A_Rocha README](../../README.md)

---

## `10.20.160.124` ù Domain Controller HIMALIA (`Jupiter.local`)

Teammate-support workspace for `.124`.

### Current evidence
- [`124-001_ldapsearch_namingcontexts_jupiter_local_dc_tm6_afrocha.png`](./Screenshots/124-001_ldapsearch_namingcontexts_jupiter_local_dc_tm6_afrocha.png)
- [`124-002_nmap_krb_smb_himalia_dc_tm6_afrocha.png`](./Screenshots/124-002_nmap_krb_smb_himalia_dc_tm6_afrocha.png)
- [`124-003_nmap_top1000_himalia_service_fingerprint_tm6_afrocha.png`](./Screenshots/124-003_nmap_top1000_himalia_service_fingerprint_tm6_afrocha.png)
- [`124-004_msf_bluekeep_show_info_targets_check_supported_tm6_afrocha.png`](./Screenshots/124-004_msf_bluekeep_show_info_targets_check_supported_tm6_afrocha.png)
- [`124-005_msf_bluekeep_options_rhosts_rport_description_tm6_afrocha.png`](./Screenshots/124-005_msf_bluekeep_options_rhosts_rport_description_tm6_afrocha.png)
- [`124-006_msf_dos_rdp_ms12_020_maxchannels_rdp_seems_down_tm6_afrocha.png`](./Screenshots/124-006_msf_dos_rdp_ms12_020_maxchannels_rdp_seems_down_tm6_afrocha.png)
- [`124-007_msf_scanner_rdp_ms12_020_check_cannot_reliably_check_tm6_afrocha.png`](./Screenshots/124-007_msf_scanner_rdp_ms12_020_check_cannot_reliably_check_tm6_afrocha.png)

### Recon + exploit hypothesis notes
- `nmap -Pn --top-ports 1000 -sV 10.20.160.124` shows a profile consistent with Windows Server 2008 R2 AD services (DNS/Kerberos/LDAP/SMB/RPC/RDP).
- BlueKeep module `exploit/windows/rdp/cve_2019_0708_bluekeep_rce` advertises support for Win7/2008 R2 families, so this is a valid **hypothesis** lane.
- MS12-020 checks in your captures are inconclusive (`rdp seems down`, `cannot reliably check exploitability`) and do not confirm vulnerability.
- This does **not** prove vulnerability by itself; confirm with safe checks and ROE before any crash-prone exploit attempt.

Use `attack_plan/README.md` for next steps and `unfruitful_attempts/README.md` for failed or inconclusive lanes.
