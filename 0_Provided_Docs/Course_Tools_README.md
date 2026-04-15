# Course tools reference (EPT Week 3 & Week 4)

This quick reference mirrors the structure used in `EPT_PWN2` and summarizes tools from:

- `Week3_EPT_Exploitation.pdf`
- `Week4_EPT_PostEx.pdf`

Use only tools allowed by your lab ROE and assignment scope.

---

## Week 3 - Exploitation workflow

| Tool / topic | General use | Typical use in this challenge |
|---|---|---|
| `msfconsole` | Exploit and auxiliary module framework | Validate and exploit discovered services |
| Meterpreter | In-memory post-exploitation session | Host enumeration and pivot support |
| `msfvenom` | Custom payload generation | Build executable payloads when needed |
| SearchSploit | Local exploit-db lookup | Validate exploit availability outside MSF |
| `autoroute` / `portfwd` | Pivoting and tunnel workflows | Reach internal assets from foothold |

---

## Week 4 - Post-exploitation workflow

| Tool / topic | General use | Typical use in this challenge |
|---|---|---|
| Windows built-ins (`whoami`, `systeminfo`, `net`) | Local recon after shell | Privilege/context discovery |
| Credential artifacts / hash collection | Proof and lateral movement prep | Capture required proof outputs |
| Privilege escalation concepts | Gain required context for proof files | Reach user/admin desktop artifacts |
| AD recon tooling (if domain exists) | Identify trust paths | Only when lab topology supports it |

---

## PWN3 framing notes

- Scope from challenge brief: `10.20.160.10-150`
- Known foothold hint: payload delivery by email to `olivia@pwn3.local`
- SMTP constraint from brief: no TLS support on available mail server
- Keep anti-cheating evidence visible in key terminals:
  - Linux: `echo <your_name>; date`
  - Windows cmd: `echo <your_name> %date% %time%`

Back to repo root: `../README.md`
