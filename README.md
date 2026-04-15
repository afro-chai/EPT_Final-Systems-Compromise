# EPT Final Systems Compromise - Team Runbook

Repository landing page for the current team workflow: scope, checkpoints, evidence standards, and report readiness.

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
| `3_Work/` | Raw notes, command logs, failed/alternate paths |
| `3_Work/<Member>/` | Per-teammate scratch (see [`3_Work/README.md`](3_Work/README.md)) |

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

- **Personal logs**: each person has a folder under `3_Work/` (`J_Solis`, `T_Amor`, `H_Padilla`, `R_White`, `B_Drummond`, `A_Rocha`) for drafts and transcripts; **promote** report-ready material to shared paths (for example `1_Screenshots/`).
- Assign one teammate as evidence curator to maintain figure numbering consistency.
- Assign one teammate as report integrator to keep section naming and tone consistent.
- Keep a single source-of-truth findings table before drafting full narrative sections.
- Track what worked and did not work so the technical story is clear and defensible.

---

## Report template

Use the included template:

- `EPT_Final_Assessment_Report.tex`

You can move it into `2_Final_Report/` if your team creates that folder.

