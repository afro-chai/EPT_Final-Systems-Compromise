# J_Solis — personal work folder

## Where to start (STEPfwd)

<table>
<tr valign="top">
<td width="62%">

If you forgot how to open the lab environment:

<ol>
<li>Log in to <strong>STEPfwd</strong> (CERT Simulation, Training &amp; Exercise platform — CMU Software Engineering Institute).</li>
<li>Open the <strong>Exercises</strong> tab (not Curriculums or Bookmarks).</li>
<li>Under Exercises, keep <strong>Exercises</strong> selected (not <strong>On Demand Labs</strong>).</li>
<li>Find the row <strong>Exercise: EPT</strong>, <strong>Session: Session 1</strong>, <strong>Team: Team_6</strong> (confirm session/team with the instructor if labels change).</li>
<li>Click <strong>Connect</strong> (the link on the right, highlighted in yellow in the screenshot) to launch the exercise and reach your assigned <strong>Kali</strong> VM.</li>
</ol>

<p><strong>This folder:</strong> use <a href="work/"><code>work/</code></a> for logs, notes, and rough output; use <a href="Screenshots/"><code>Screenshots/</code></a> for draft captures. Promote <strong>final</strong> evidence to the team folder <a href="../1_Screenshots/"><code>1_Screenshots/</code></a> when it is report-ready.</p>

</td>
<td width="38%" align="center">

<img src="Screenshots/stepfwd_connect_team6.png" width="280" alt="STEPfwd dashboard: Exercises tab, EPT Session 1, Team_6, yellow Connect link"/>

<p><small>STEPfwd UI (scaled for README)</small></p>

</td>
</tr>
</table>

---

## Host_Discovery (master)

**Edit with the team and keep in sync with the root README host table:**

[Host_Discovery — Google Sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing)

**Default rule:** everyone should own **at least three** IPs in the lab range. You can **trade** hosts by updating the sheet, then mirror changes in [root `README.md`](../README.md) and each affected teammate’s `README.md`.

**Your Kali VM:** **KALI1** (same as PWN2). Use this machine for scans, exploits, and listeners so teammates do not overwrite each other’s work.

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
| | `10.20.160.126` | | | |
| | `10.20.160.125` | 135; 139; 445; 49152; 49153; 49154; 49182; 49186 | | |
| | `10.20.160.124` | 135; 139; 389; 445; 3268; 5722; 49152; 49153; 49154; 49155; 49158; 49179; 49193; 49199 | | |

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
