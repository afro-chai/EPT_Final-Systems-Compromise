# EPT Final Systems Compromise - Team Runbook

Repository landing page for the current team workflow: scope, host tracking, per-member checkpoints, and report readiness.

## Host discovery (tracking)

Snapshot from the team **Host_Discovery** sheet. Scan columns show **Complete** where Drive links existed (replace with real links when ready); otherwise leave blank until uploaded.

| Exploited? | IP | Ports | Network mapping scan | Vulnerability scan |
|------------|-----|-------|----------------------|--------------------|
| | `10.20.160.126` | | | |
| | `10.20.160.125` | 135; 139; 445; 49152; 49153; 49154; 49182; 49186 | | |
| | `10.20.160.124` | 135; 139; 389; 445; 3268; 5722; 49152; 49153; 49154; 49155; 49158; 49179; 49193; 49199 | | |
| | `10.20.160.123` | | | |
| | `10.20.160.122` | 135; 139; 445; 49152; 49153; 49154; 49155; 49211; 49212 | Complete | Complete |
| | `10.20.160.112` | | Complete | Complete |
| | `10.20.160.111` | | | |
| | `10.20.160.110` | 135; 139; 445; 49152; 49153; 49154; 49179; 49187 | | |
| | `10.20.160.109` | 135; 139; 445; 49152; 49153; 49154; 49174; 49187 | | |
| | `10.20.160.108` | 139; 445 | | |
| | `10.20.160.107` | | | |
| | `10.20.160.106` | 135; 445; 49152; 49153; 49154; 49180; 49182 | | |
| | `10.20.160.105` | 135; 139; 445; 49152; 49153; 49154; 49178; 49189 | | |
| | `10.20.160.104` | 135; 49152; 49153; 49154; 49155; 49182 | | |
| | `10.20.160.103` | 21; 80; 443; 3306 | Complete | Complete |
| | `10.20.160.102` | | | |
| | `10.20.160.101` | 135; 139; 445; 49152; 49153; 49154; 49155; 49217; 49222 | | |
| | `10.20.160.100` | 139; 445 | | |

---

## Team context

- Project type: **team-based final assessment**
- Course: **Ethical Penetration Testing**
- Deliverables:
  - Final technical report (LaTeX template included)
  - Evidence screenshots with captions
  - Structured findings by host

---

## Suggested repository layout

| Path | Purpose |
|------|---------|
| [`README.md`](README.md) | Team runbook, host discovery table, checkpoints |
| [`0_Provided_Docs/`](0_Provided_Docs/) | Course PDFs, rubric, countdown (local copy if present) |
| [`1_Screenshots/`](1_Screenshots/) | Report-ready figures (create when you start capturing) |
| [`2_Final_Report/`](2_Final_Report/) | Final PDF source (optional folder; template can live here) |
| [`EPT_Final_Assessment_Report.tex`](EPT_Final_Assessment_Report.tex) | LaTeX report template (move under `2_Final_Report/` if you prefer) |
| [`J_Solis/`](J_Solis/) | J_Solis scratch + personal CP checklist (below) |
| [`T_Amor/`](T_Amor/) | T_Amor scratch + personal CP checklist (below) |
| [`H_Padilla/`](H_Padilla/) | H_Padilla scratch + personal CP checklist (below) |
| [`R_White/`](R_White/) | R_White scratch + personal CP checklist (below) |
| [`B_Drummond/`](B_Drummond/) | B_Drummond scratch + personal CP checklist (below) |
| [`A_Rocha/`](A_Rocha/) | A_Rocha scratch + personal CP checklist (below) |

---

## Scope and guardrails

- Stay within the instructor-authorized lab scope only.
- Record enough detail for reproducibility (host, service, command purpose, result).
- Keep evidence ethical and course-compliant (no out-of-scope testing).

---

## Checkpoint advice (team)

Use this as the **default ladder**; split hosts and roles in stand-up so nobody duplicates work without logging it.

- **CP0 — Setup:** Confirm authorized range, folder conventions, and who owns which IPs or workstreams. Log identity stamps if your workflow uses them.
- **CP1 — Discovery:** Keep the [Host discovery](#host-discovery-tracking) table honest (IPs, ports, scan column placeholders). Dead hosts and empty rows are fine if documented.
- **CP2 — Mapping:** For each interesting host, note services, likely attack surface, and what you ruled out (not only what worked).
- **CP3 — Exploitation:** Time-box attempts; capture enough context that someone else can reproduce or understand a negative result.
- **CP4 — Proof:** Treat `local.txt` / `proof.txt` (or equivalent) as explicit gates; screenshot or transcribe with timestamps where required.
- **CP5 — Lateral / pivot:** Only after CP4 is satisfied or explicitly blocked and documented; note pivot path and credentials involved (no secrets in git).
- **CP6 — Report pass:** One person should own merge conflicts on the final `.tex` / doc; everyone else supplies figures and bullet findings.
- **CP7 — Submit:** Final read against rubric, figure order matches narrative, repo clean, submission package complete.

**Roles (pick names, not everyone on everything):** evidence / figure numbering owner, report integrator, “negative path” logger so the story is defensible.

---

## Checkpoints by teammate folder

Same CP ladder in each block; tick in your folder’s `README.md` or here as you go. **Promote** report-ready screenshots into `1_Screenshots/`; keep rough logs under your name.

### [`J_Solis/`](J_Solis/)

- [ ] **CP0–CP1:** Scope + discovery notes; update shared host table if you touched an IP.
- [ ] **CP2–CP3:** Vuln hypotheses and exploit attempts (include failures).
- [ ] **CP4–CP5:** Proof / pivot artifacts or explicit “blocked” notes.
- [ ] **CP6–CP7:** Figures and text handed to report owner; submission-ready pass.

### [`T_Amor/`](T_Amor/)

- [ ] **CP0–CP1:** Scope + discovery notes; update shared host table if you touched an IP.
- [ ] **CP2–CP3:** Vuln hypotheses and exploit attempts (include failures).
- [ ] **CP4–CP5:** Proof / pivot artifacts or explicit “blocked” notes.
- [ ] **CP6–CP7:** Figures and text handed to report owner; submission-ready pass.

### [`H_Padilla/`](H_Padilla/)

- [ ] **CP0–CP1:** Scope + discovery notes; update shared host table if you touched an IP.
- [ ] **CP2–CP3:** Vuln hypotheses and exploit attempts (include failures).
- [ ] **CP4–CP5:** Proof / pivot artifacts or explicit “blocked” notes.
- [ ] **CP6–CP7:** Figures and text handed to report owner; submission-ready pass.

### [`R_White/`](R_White/)

- [ ] **CP0–CP1:** Scope + discovery notes; update shared host table if you touched an IP.
- [ ] **CP2–CP3:** Vuln hypotheses and exploit attempts (include failures).
- [ ] **CP4–CP5:** Proof / pivot artifacts or explicit “blocked” notes.
- [ ] **CP6–CP7:** Figures and text handed to report owner; submission-ready pass.

### [`B_Drummond/`](B_Drummond/)

- [ ] **CP0–CP1:** Scope + discovery notes; update shared host table if you touched an IP.
- [ ] **CP2–CP3:** Vuln hypotheses and exploit attempts (include failures).
- [ ] **CP4–CP5:** Proof / pivot artifacts or explicit “blocked” notes.
- [ ] **CP6–CP7:** Figures and text handed to report owner; submission-ready pass.

### [`A_Rocha/`](A_Rocha/)

- [ ] **CP0–CP1:** Scope + discovery notes; update shared host table if you touched an IP.
- [ ] **CP2–CP3:** Vuln hypotheses and exploit attempts (include failures).
- [ ] **CP4–CP5:** Proof / pivot artifacts or explicit “blocked” notes.
- [ ] **CP6–CP7:** Figures and text handed to report owner; submission-ready pass.

---

## Report template

Use the included template:

- `EPT_Final_Assessment_Report.tex`

You can move it into `2_Final_Report/` if your team creates that folder.

