# Course tools reference (EPT Week 3 & Week 4 slides)

Summarizes tools and concepts from the PDFs in this folder:

- **`Week3_EPT_Exploitation.pdf`** — exploitation / Metasploit
- **`Week4_EPT_PostEx.pdf`** — post-exploitation, payloads, AD

Your lab image may not include every binary; use only what ROE and the VM provide.

---

## Week 3 — Ethical exploitation (Metasploit era)

| Tool / topic | Generally | Good for |
|--------------|-----------|----------|
| **Metasploit Framework** (`msfconsole`) | Modular platform: exploits, **auxiliary** scanners, **post** modules, **payloads**, encoders, evasion | Turning a known vuln + target service into a session; `search` / `use` / `set` / `exploit` |
| **Meterpreter** | Staged in-memory payload | Stable foothold, file ops, post modules |
| **`msfvenom`** | Standalone payload generator | Custom **exe**, **php**, **raw**, etc. |
| **`autoroute` + pivoting** | Route through compromised host | Internal subnets |
| **`portfwd`** | TCP forward via Meterpreter | Tunnel to internal services |
| **`run netenum`** (Meterpreter) | Ping sweep from compromised host | Inside-out discovery |
| **Armitage** | Metasploit GUI / collaboration | Same backend as MSF |
| **Empire** / **Cobalt Strike** | Advanced C2 | Only if lab explicitly allows |

---

## Week 4 — Post-exploitation, payloads, AD

| Tool / topic | Generally | Good for |
|--------------|-----------|----------|
| **ExploitDB / SearchSploit** | Public exploits + CLI search | When MSF has no module; review code first |
| **Payload concepts** (stager, shellcode, bind vs reverse) | How shells work | Choosing payloads, badchars |
| **`msfvenom`** (flags) | `-p`, `-f`, `-e`, `-i`, `-a`, `-b` | Tuning output |
| **Veil / Veil-Evasion** | Obfuscated payloads | AV evasion (per ROE) |
| **Windows enumeration** (`whoami`, `ipconfig`, `net`, `systeminfo`, …) | Built-in recon | After foothold |
| **Easy-win searches** (`dir /s`, `findstr`, `reg query`, unattended XML) | Loose creds | Quick wins |
| **Privilege escalation** | `getsystem`, local exploits, misconfigs | **proof.txt** often needs admin |
| **GPP / credentials** | SYSVOL `Groups.xml` style | AD lateral movement |
| **Hash dump / Mimikatz** | NTLM / memory creds | PTH, lateral movement |
| **CrackMapExec** | AD automation | If installed + allowed |
| **Persistence** (registry, **schtasks**, Startup) | Course topic | Only if ROE permits |
| **PowerSploit** (PowerUp, PowerView) | PS privesc / AD recon | Misconfigs, shares |
| **BloodHound** | AD attack-path graphs | Complex domains |

---

## Initial bias for PWN Challenge #2 (not a recipe)

1. Map **`10.20.160.10–200`**, match services to **Metasploit** + **SearchSploit**.
2. **`msfconsole`** + exploit + **Meterpreter** (or shell) → enumerate.
3. **`msfvenom`** / **Veil** if you need custom or evasive payloads (per ROE).
4. **Pivot** with **Meterpreter** if targets are not directly reachable.
5. Plan **privesc** if **proof.txt** is on an admin desktop.
6. AD tools (**GPP**, **Mimikatz**, **BloodHound**, …) only when the lab is domain-joined and tools exist.

---

Back to project root: [`../README.md`](../README.md) · Session / try log: [`../3_Work/README.md`](../3_Work/README.md)
