# Team work area (`3_Work/`)

Shared space for **raw notes**, **command transcripts**, **things that did not work**, and **session scratch** before anything is promoted into the final report or `1_Screenshots/`.

## Why folders instead of one branch per person

- **One branch (`main`)** keeps the report, evidence list, and findings in one place so nobody merges stale work at the deadline.
- **Per-member folders** give everyone a clear place to dump logs without stepping on filenames, with low coordination overhead.
- Use **short-lived branches** only if two people need to edit the same `.tex` file at once and want review before merge; day-to-day notes belong here.

## Member folders

| Folder | Use |
|--------|-----|
| [`J_Solis/`](J_Solis/) | Personal notes, command logs, drafts |
| [`T_Amor/`](T_Amor/) | Personal notes, command logs, drafts |
| [`H_Padilla/`](H_Padilla/) | Personal notes, command logs, drafts |
| [`R_White/`](R_White/) | Personal notes, command logs, drafts |
| [`B_Drummond/`](B_Drummond/) | Personal notes, command logs, drafts |
| [`A_Rocha/`](A_Rocha/) | Personal notes, command logs, drafts |

## Conventions

- **Promote to shared artifacts**: put final, report-ready screenshots under `1_Screenshots/` (or whatever the team agrees) with consistent naming; keep rough drafts in your folder.
- **Avoid secrets in git**: if you paste real passwords or tokens by mistake, rotate them and scrub history per course/instructor guidance.
- **Cross-link**: when something in your folder becomes official, add a one-line pointer in the main `README.md` or the report so graders can follow the story.
