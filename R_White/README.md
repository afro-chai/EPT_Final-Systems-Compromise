# R_White — personal work folder

## Where to start (STEPfwd)

If you forgot how to open the lab environment:

1. Log in to **STEPfwd** (CERT Simulation, Training & Exercise platform — CMU Software Engineering Institute).
2. Open the **Exercises** tab (not Curriculums or Bookmarks).
3. Under Exercises, keep **Exercises** selected (not **On Demand Labs**).
4. Find the row **Exercise: EPT**, **Session: Session 1**, **Team: Team_6** (confirm session/team with the instructor if labels change).
5. Click **Connect** (the link on the right, often highlighted) to launch the exercise and reach your assigned **Kali** VM.

![STEPfwd: Exercises → EPT Session 1 → Team_6 → Connect](../0_Provided_Docs/STEPfwd_EPT_Session1_Team6_connect.png)

**This folder:** use [`work/`](work/) for logs, notes, and rough output; use [`Screenshots/`](Screenshots/) for draft captures. Promote **final** evidence to the team folder [`../1_Screenshots/`](../1_Screenshots/) when it is report-ready.

---

## Host_Discovery (master)

**Edit with the team and keep in sync with the root README host table:**

[Host_Discovery — Google Sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing)

**Default rule:** everyone should own **at least three** IPs in the lab range. You can **trade** hosts by updating the sheet, then mirror changes in [root `README.md`](../README.md) and each affected teammate’s `README.md`.

**Your Kali VM:** **KALI4** (same as PWN2). Use this machine for scans, exploits, and listeners so teammates do not overwrite each other’s work.

| Teammate | Kali VM |
|----------|---------|
| J_Solis | **KALI1** |
| T_Amor | **KALI2** |
| H_Padilla | **KALI3** |
| R_White | **KALI4** |
| B_Drummond | **KALI5** |
| A_Rocha | **KALI6** |

---

## Your hosts (you are responsible for these by default)

| Exploited? | IP | Ports | Network mapping scan | Vulnerability scan |
|------------|-----|-------|----------------------|--------------------|
| | `10.20.160.108` | 139; 445 | | |
| | `10.20.160.107` | | | |
| | `10.20.160.106` | 135; 445; 49152; 49153; 49154; 49180; 49182 | | |

---

## Checkpoints (CP)

| CP | Gate |
|----|------|
| CP0 | Setup: authorized scope, folder habit, confirm or swap IP ownership with team |
| CP1 | Discovery: your rows + sheet + root README stay aligned |
| CP2 | Mapping: services, attack surface, ruled-out paths for your IPs |
| CP3 | Exploitation: attempts documented (success or failure) |
| CP4 | Proof: `local.txt` / `proof.txt` (or equivalent) or explicit “blocked” |
| CP5 | Lateral / pivot if in scope; never commit secrets |
| CP6 | Report: hand figures and narrative snippets to integrator |
| CP7 | Submit: final checklist and rubric pass |

**Track here (GitHub checkboxes):**

- [ ] CP0 — Setup / ownership clear
- [ ] CP1 — Discovery updated for my IPs
- [ ] CP2 — Mapping notes for my IPs
- [ ] CP3 — Exploitation attempts logged
- [ ] CP4 — Proof or documented block
- [ ] CP5 — Lateral / pivot (if applicable)
- [ ] CP6 — Report handoff done
- [ ] CP7 — Submit-ready
