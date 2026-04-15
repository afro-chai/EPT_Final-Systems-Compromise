# EPT Final Systems Compromise - Team Runbook

Repository landing page for the current team workflow: scope, checkpoints, evidence standards, and report readiness.

## Host discovery (tracking)

Snapshot from the team **Host_Discovery** sheet. Scan columns show **Complete** where Drive links existed (replace with real links when ready); otherwise leave blank until uploaded.

| Reported? | IP | Ports | Network mapping scan | Vulnerability scan |
|-----------|-----|-------|----------------------|--------------------|
| | `10.20.160.126` | | | |
| | `10.20.160.125` | 135; 139; 445; 49152; 49153; 49154; 49182; 49186 | | |
| | `10.20.160.124` | 135; 139; 389; 445; 3268; 5722; 49152; 49153; 49154; 49155; 49158; 49179; 49193; 49199 | | |
| | `10.20.160.123` | | | |
| YES / HW2 / AFR | `10.20.160.122` | 135; 139; 445; 49152; 49153; 49154; 49155; 49211; 49212 | Complete | Complete |
| YES / HW2 / AFR | `10.20.160.112` | | Complete | Complete |
| | `10.20.160.111` | | | |
| | `10.20.160.110` | 135; 139; 445; 49152; 49153; 49154; 49179; 49187 | | |
| | `10.20.160.109` | 135; 139; 445; 49152; 49153; 49154; 49174; 49187 | | |
| | `10.20.160.108` | 139; 445 | | |
| | `10.20.160.107` | | | |
| | `10.20.160.106` | 135; 445; 49152; 49153; 49154; 49180; 49182 | | |
| | `10.20.160.105` | 135; 139; 445; 49152; 49153; 49154; 49178; 49189 | | |
| | `10.20.160.104` | 135; 49152; 49153; 49154; 49155; 49182 | | |
| YES / HW2 / AFR | `10.20.160.103` | 21; 80; 443; 3306 | Complete | Complete |
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
| `README.md` | Main team runbook and grading alignment |
| `0_Provided_Docs/` | Course docs, countdown notes, rubric |
| `1_Screenshots/` | Labeled figures used in the report |
| `2_Final_Report/` | Final report source and export |
| `3_Work/` | Shared raw notes, command logs, failed/alternate paths ([`3_Work/README.md`](3_Work/README.md)) |
| `J_Solis/`, `T_Amor/`, `H_Padilla/`, `R_White/`, `B_Drummond/`, `A_Rocha/` | Per-teammate scratch at repo root |

---

## Scope and guardrails

- Stay within the instructor-authorized lab scope only.
- Record enough detail for reproducibility (host, service, command purpose, result).
- Keep evidence ethical and course-compliant (no out-of-scope testing).

---

## Team workflow checkpoints

| CP | Gate | Status |
|----|------|--------|
| CP0 | Confirm scope + team task split | In progress |
| CP1 | Discovery and host inventory | In progress |
| CP2 | Vulnerability analysis by host | Pending |
| CP3 | Exploitation and access validation | Pending |
| CP4 | Proof collection (`local.txt` / `proof.txt`) | Pending |
| CP5 | Lateral movement / pivot validation (if in-scope) | Pending |
| CP6 | Final report quality pass | Pending |
| CP7 | Submission checklist closure | Pending |

---

## Required report section names (exact)

Use these section titles exactly to match grading expectations:

1. `Executive Summary`
2. `Detailed Findings`
3. `Technical Overview`
4. `Technical Details`

Also include:

- Title page
- Table of contents
- Page numbers
- Numbered figure captions

---

## Evidence standards

- Every major claim should map to at least one screenshot or command output artifact.
- Label figures sequentially (Figure 1, Figure 2, ...).
- Keep screenshot names descriptive and consistent, for example:
  - `01_scope_discovery.png`
  - `02_host_services_<host>.png`
  - `03_exploit_success_<host>.png`
  - `04_proof_file_<host>.png`

---

## Detailed findings format (per finding)

For each finding, always include:

- Vulnerability name
- Description
- Severity (Critical / High / Medium / Low)
- Affected host(s)
- Impact
- Recommended mitigation
- Evidence reference (Figure #)

---

## Team execution notes

- **Personal logs**: each person has a folder at repo root (`J_Solis/`, `T_Amor/`, `H_Padilla/`, `R_White/`, `B_Drummond/`, `A_Rocha/`) for drafts and transcripts; **promote** report-ready material to shared paths (for example `1_Screenshots/`).
- Assign one teammate as evidence curator to maintain figure numbering consistency.
- Assign one teammate as report integrator to keep section naming and tone consistent.
- Keep a single source-of-truth findings table before drafting full narrative sections.
- Track what worked and did not work so the technical story is clear and defensible.

---

## Report template

Use the included template:

- `EPT_Final_Assessment_Report.tex`

You can move it into `2_Final_Report/` if your team creates that folder.

