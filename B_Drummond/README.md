# B_Drummond — personal work folder

## Host_Discovery (master)

**Edit with the team and keep in sync with the root README host table:**

[Host_Discovery — Google Sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing)

**Default rule:** everyone should own **at least three** IPs in the lab range. You can **trade** hosts by updating the sheet, then mirror changes in [root `README.md`](../README.md) and each affected teammate’s `README.md`.

Rough notes and command logs stay here. Move **final** evidence to [`../1_Screenshots/`](../1_Screenshots/) when it is report-ready.

---

## Your hosts (you are responsible for these by default)

| Exploited? | IP | Ports | Network mapping scan | Vulnerability scan |
|------------|-----|-------|----------------------|--------------------|
| | `10.20.160.105` | 135; 139; 445; 49152; 49153; 49154; 49178; 49189 | | |
| | `10.20.160.104` | 135; 49152; 49153; 49154; 49155; 49182 | | |
| | `10.20.160.103` | 21; 80; 443; 3306 | Complete | Complete |

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
