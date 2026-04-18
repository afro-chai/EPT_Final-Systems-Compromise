# EPT Final Systems Compromise - Team Runbook

Repository landing page for the current team workflow: scope, host tracking, per-member checkpoints, and report readiness.

## Due dates (Canvas, chronological)

Times are as listed on Canvas (confirm in Canvas if your section differs). **2026** dates below.

| Due | Deliverable | Points | Submission |
|-----|-------------|--------|--------------|
| **Wed Apr 22, 2026 · 6:30 PM** | Final Presentation | 30 | Team submission on Canvas |
| **Sun Apr 26, 2026 · 6:30 PM** | Final Report | 50 | No separate Canvas upload for this line item; **50 pts** from the submitted report |
| **Sun Apr 26, 2026 · 11:59 PM** | Final Systems Compromised | 10 | No separate Canvas upload; **10 pts** from report / engagement evidence |
| **Mon Apr 27, 2026 · 11:59 PM** | Peer Review (quiz) | 10 | **Individual** Canvas submission |

---

## Host discovery (tracking)

**Live sheet (single source of truth for scans and handoffs):**  
[Host_Discovery — Google Sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing)

**Lab Kali (PWN2 convention):** Everyone should use the **same Kali VM from PWN2** for this engagement so we do not collide on shells, `LHOST` / listener ports, Metasploit workspaces, or home-directory scan output. Stay on your assigned machine unless the team explicitly agrees a swap.

| Teammate | Kali VM | Responsibilities |
|----------|---------|------------------|
| J_Solis | **KALI1** | **Presentation** & team **submission** (final deck / Canvas for the presentation deliverable) |
| T_Amor | **KALI2** | **Presentation** & team **submission** (final deck / Canvas for the presentation deliverable) |
| H_Padilla | **KALI3** | **Coordination** & **synchronization** (stand-ups, handoffs, sheet/repo alignment—pair with A_Rocha) |
| R_White | **KALI4** | **Written report** & team **submission** (LaTeX/PDF, Canvas for the report deliverable) |
| B_Drummond | **KALI5** | **Written report** & team **submission** (LaTeX/PDF, Canvas for the report deliverable) |
| A_Rocha | **KALI6** | **Coordination** & **synchronization** (stand-ups, handoffs, sheet/repo alignment—pair with H_Padilla) |

**Lab portal (STEPfwd):** step-by-step **Connect** instructions and screenshot live at the top of each teammate’s folder README and in [`0_Provided_Docs/README.md`](0_Provided_Docs/README.md).

The table below mirrors the sheet for GitHub. **Scan columns:** use **Complete** where Drive links exist in the sheet (swap in markdown links here if you want them in-repo). **Responsible:** default is **three IPs per teammate** (top-to-bottom list); trade in the sheet and update this table + the owner’s folder `README.md` if you reassign.

| Exploited? | Responsible | IP | Ports | Network mapping scan | Vulnerability scan |
|------------|---------------|-----|-------|----------------------|--------------------|
| | **J_Solis** | `10.20.160.126` | 22; 25; 110; 143 | | |
| | **J_Solis** | `10.20.160.125` | 135; 139; 445; 49152; 49153; 49154; 49182; 49186 | | |
| | **J_Solis** | `10.20.160.124` | 135; 139; 389; 445; 3268; 5722; 49152; 49153; 49154; 49155; 49158; 49179; 49193; 49199 | | |
| | **T_Amor** | `10.20.160.123` | 53, 80| | |
| | **T_Amor** | `10.20.160.122` | 135; 139; 445; 49152; 49153; 49154; 49155; 49211; 49212 | Complete | Complete |
| | **T_Amor** | `10.20.160.112` | 22, 80| Complete | Complete |
| | **H_Padilla** | `10.20.160.111` | | | |
| Exploited | **H_Padilla** | `10.20.160.110` | 135; 139; 445; 49152; 49153; 49154; 49179; 49187 | Complete | Complete |
| Exploited | **H_Padilla** | `10.20.160.109` | 135; 139; 445; 49152; 49153; 49154; 49174; 49187 | Complete | Complete |
| | **R_White** | `10.20.160.108` | 139; 445 | | |
| | **R_White** | `10.20.160.107` | | | |
| | **R_White** | `10.20.160.106` | 135; 445; 49152; 49153; 49154; 49180; 49182 | | |
| | **B_Drummond** | `10.20.160.105` | 135; 139; 445; 49152; 49153; 49154; 49178; 49189 | | |
| | **B_Drummond** | `10.20.160.104` | 135; 49152; 49153; 49154; 49155; 49182 | | |
| | **B_Drummond** | `10.20.160.103` | 21; 80; 443; 3306 | Complete | Complete |
| | **A_Rocha** | `10.20.160.102` | | | |
| | **A_Rocha** | `10.20.160.101` | 135; 139; 445; 49152; 49153; 49154; 49155; 49217; 49222 | | |
| | **A_Rocha** | `10.20.160.100` | 139; 445 | | |

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
| [`0_Provided_Docs/`](0_Provided_Docs/) | Course PDFs, rubric, slide screenshots — start at [`0_Provided_Docs/README.md`](0_Provided_Docs/README.md) |
| [`1_Screenshots/`](1_Screenshots/) | Report-ready figures (create when you start capturing) |
| [`2_Final_Report/`](2_Final_Report/) | Final PDF source (optional folder; template can live here) |
| [`EPT_Final_Assessment_Report.tex`](EPT_Final_Assessment_Report.tex) | LaTeX report template (move under `2_Final_Report/` if you prefer) |
| [`J_Solis/`](J_Solis/) | README (STEPfwd, Kali, hosts, CP), [`work/`](J_Solis/work/), [`Screenshots/`](J_Solis/Screenshots/) |
| [`T_Amor/`](T_Amor/) | README (STEPfwd, Kali, hosts, CP), [`work/`](T_Amor/work/), [`Screenshots/`](T_Amor/Screenshots/) |
| [`H_Padilla/`](H_Padilla/) | README (STEPfwd, Kali, hosts, CP), [`work/`](H_Padilla/work/), [`Screenshots/`](H_Padilla/Screenshots/) |
| [`R_White/`](R_White/) | README (STEPfwd, Kali, hosts, CP), [`work/`](R_White/work/), [`Screenshots/`](R_White/Screenshots/) |
| [`B_Drummond/`](B_Drummond/) | README (STEPfwd, Kali, hosts, CP), [`work/`](B_Drummond/work/), [`Screenshots/`](B_Drummond/Screenshots/) |
| [`A_Rocha/`](A_Rocha/) | README (STEPfwd, Kali, hosts, CP), [`work/`](A_Rocha/work/), [`Screenshots/`](A_Rocha/Screenshots/) |

---

## Scope and guardrails

- Stay within the instructor-authorized lab scope only.
- Record enough detail for reproducibility (host, service, command purpose, result).
- Keep evidence ethical and course-compliant (no out-of-scope testing).

---

## Checkpoint advice (team)

Use this as the **default ladder**; split hosts and roles in stand-up so nobody duplicates work without logging it.

- **CP0 — Setup:** Confirm authorized range, folder conventions, and who owns which IPs or workstreams. Log identity stamps if your workflow uses them.
- **CP1 — Discovery:** Keep the [Host discovery](#host-discovery-tracking) table and [sheet](https://docs.google.com/spreadsheets/d/1J3H6ee8N06ROz1l4pcgxclArq5TN4sk3kIPbHPTCj3A/edit?usp=sharing) aligned (IPs, ports, **Responsible**, scan placeholders).
- **CP2 — Mapping:** For each interesting host, note services, likely attack surface, and what you ruled out (not only what worked).
- **CP3 — Exploitation:** Time-box attempts; capture enough context that someone else can reproduce or understand a negative result.
- **CP4 — Proof:** Treat `local.txt` / `proof.txt` (or equivalent) as explicit gates; screenshot or transcribe with timestamps where required.
- **CP5 — Lateral / pivot:** Only after CP4 is satisfied or explicitly blocked and documented; note pivot path and credentials involved (no secrets in git).
- **CP6 — Report pass:** One person should own merge conflicts on the final `.tex` / doc; everyone else supplies figures and bullet findings.
- **CP7 — Submit:** Final read against rubric, figure order matches narrative, repo clean, submission package complete.

**Roles (pick names, not everyone on everything):** evidence / figure numbering owner, report integrator, “negative path” logger so the story is defensible.

---

## Checkpoints by teammate folder

Each teammate folder has its own **CP table** and **IP slice** (three hosts by default). Tick boxes there or in the sheet; keep this README’s host table aligned when you trade rows.

| Folder | README |
|--------|--------|
| J_Solis | [`J_Solis/README.md`](J_Solis/README.md) |
| T_Amor | [`T_Amor/README.md`](T_Amor/README.md) |
| H_Padilla | [`H_Padilla/README.md`](H_Padilla/README.md) |
| R_White | [`R_White/README.md`](R_White/README.md) |
| B_Drummond | [`B_Drummond/README.md`](B_Drummond/README.md) |
| A_Rocha | [`A_Rocha/README.md`](A_Rocha/README.md) |

---

## Report template

The final write-up should follow the course report structure (Executive Summary, Detailed Findings, Technical Overview, Technical Details, plus title page, TOC, page numbers, and numbered figures). Use one of these workflows—pick what the **report integrator** prefers and stick to it:

| Where | Use for |
|-------|---------|
| **[Team report on Overleaf](https://www.overleaf.com/2326867146vjjczghdjbdx#5c8013)** | Shared editing, PDF build, and submission packaging. Invite collaborators from Overleaf’s **Share** menu (same emails as GitHub/Drive helps). |
| **[`EPT_Final_Assessment_Report.tex`](EPT_Final_Assessment_Report.tex)** (this repo) | Version-controlled source; copy changes to or from Overleaf when sections stabilize, or compile locally with `pdflatex` if you skip Overleaf. |

**Handy split:** keep evidence and narrative drafts in Git + member folders; treat Overleaf as the **integration + PDF** lane unless you fully compile from the repo.

If you want a dedicated folder in Git, move or duplicate the `.tex` (and figures path) under [`2_Final_Report/`](2_Final_Report/) and point Overleaf at a synced project or manual upload—just avoid two divergent “final” copies without an owner.

---

## Team Drive (final deck + archive)

Shared **Google Drive** for the **final presentation** and **historical** team work (slides, drafts, etc.):

[EPT Final Project — Google Drive folder](https://drive.google.com/drive/folders/1HmTMpzZdm0Z97udgU8P0C70jRPTnjny7?usp=sharing)

---

## Inviting teammates

**GitHub (this repo)** — someone with **admin** on the repository: **Settings → Collaborators** (or **Collaborators and teams** on an org repo) → **Add people**, enter their GitHub username or email, choose role (**Write** is typical for teammates), and send the invite. They must **accept** the invitation (email or GitHub notifications) before they can push.

**Google Drive (folder above)** — open the folder in Drive → **Share** → add emails and set access (**Editor** if they should upload the deck, **Viewer** if read-only). For a class team, **Editor** for all presenters usually avoids last-minute upload blocks.
