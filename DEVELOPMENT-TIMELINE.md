# Development Timeline: `ARCHIVE-DVD-VIDEO.zsh`

A history of the main script, built from the commit history of the `main` and `legacy` branches of [tomackroyd/ARCHIVE-DVD-VIDEO](https://github.com/tomackroyd/ARCHIVE-DVD-VIDEO).

The script has been renamed and replaced along the way, so this timeline follows the script's role rather than one filename. `legacy` splits off `main` at `1b319e7` (23 Oct 2025) and has two commits of its own.

**Key:**
- 🤖 marks commits with a `Co-Authored-By: Claude …` trailer.
- 🌿 marks commits that exist only on `legacy`.

## When Claude Code started

**27 Jan 2026, 18:24 NZDT: commit `821a607`, "Add improved version of ddrescue script with enhanced error handling".**

This is the first commit with a Claude co-author trailer. Before it, every commit was either made in the GitHub web UI ("Add files via upload", "Update README.md") or a short hand-written message. From this point on, commits come from a local git identity and have long, structured messages. In all, 29 commits carry a Claude trailer:

- **Claude Sonnet 4.5**: 27 Jan to 20 Feb 2026 (14 commits)
- **Claude Sonnet 4.6**: 27 Feb to 9 Apr 2026 (15 commits)

---

## Phase 1: Written by hand (May–Oct 2025)

- **31 May 2025, `16156e2`**: `ARCHIVE DVD-VIDEO.zsh` is created (256 lines). It images with `dd`, extracts with `makemkvcon --minlength=15`, and makes bwdif-deinterlaced H.264 access files from a numbered menu.
- **16 Oct 2025**: Log handling and exit logic are refactored.
- **23 Oct 2025, `5538b81` → `1b319e7`**: A day of fixing exit errors. A `_WITH_MKV_JSON` variant is added, and JSON creation is removed from the main script. **This is where `legacy` later branches off.**

## Phase 2: ddrescue, and Claude Code starts (Jan 2026)

- **21 Jan 2026, `6530467`**: `ARCHIVE DVD-VIDEO_ddrescue.zsh` is uploaded through the web UI, replacing `dd` with `ddrescue -r3`. This change was made before Claude Code.
- **27 Jan, `821a607` 🤖: Claude Code is used for the first time.** It adds `_ddrescue_improved.zsh` with better error handling.
- **28 Jan, `d0ff0a8` 🤖**: A 5% safety margin is added to the disc size passed to ddrescue.
- **28 Jan, `7f5d561` 🤖**: The README is completely rewritten. It's reverted and reworked that afternoon (`3fa2b92`, `d3a8594`).
- **28 Jan, `333fb8d`**: The original `dd` scripts are removed from `main`.
- **28 Jan, `1ba3e2a` 🤖**: The two production scripts are added: `ARCHIVE DVD-VIDEO.zsh` (blend) and **`_HOME.zsh` (bob), the ancestor of today's main script**. Both add dependency checks and error handling around each tool call.
- **28 Jan, `876326a` 🌿**: The ddrescue prototypes are archived to `legacy`. The archived "improved" script is identical to `d0ff0a8`.
- **28 Jan, `611acf8`**: The prototypes are removed from `main`, and the message points to the `legacy` branch.

## Phase 3: Variants, then one script (Jan–Feb 2026, Sonnet 4.5)

- **29 Jan, `46edbfb` 🤖**: The MEZZANINE (ProRes) variant is added.
- **30 Jan**: Files are renamed through the web UI. `_HOME` becomes **`ARCHIVE-DVD-VIDEO.zsh`**, and the blend version becomes `-BLEND`.
- **2 Feb, `953c087`**: MakeMKV `--minlength` drops from 15 to 5 seconds (manual edit).
- **3 Feb, `bddc669`, `41939ba`**: The `-O.iso` suffix is tried and reverted (manual).
- **4 Feb, `cabee92` 🌿**: `legacy`'s README is edited to remove a note about logging in. This is the last activity on that branch.
- **12 Feb, `4187a65` 🤖**: Input handling is hardened with `read -r` and device validation.
- **13 Feb, `1b631a5`, `09650ce` 🤖**: Duration-based extraction validation is added and drag-and-drop paths are fixed. The troubleshooting guide and README are updated the same day.
- **13 Feb, `a96d7d0` 🤖**: `CLAUDE.md` is added to `.gitignore`. This shows a project-level Claude setup existed by then.
- **20 Feb, `c17bf82` 🤖**: Logs are renamed in the ISO-only workflow.
- **20 Feb, `fc1a6a4` 🤖**: **The BLEND and MEZZANINE variants are removed**, leaving one script.

## Phase 4: Sonnet 4.6 and the cell-removal approach (Feb–Mar 2026)

- **27 Feb, `cd482aa` 🤖**: The first Sonnet 4.6 commit wraps ddrescue in `caffeinate -dims`.
- **2–5 Mar 🤖**: Two docs are written with Claude: an imaging guide for non-technical operators and the Sony recorder authoring guide.
- **26 Mar**: The git identity changes from `twelveplusone` to `tomackroyd`. The script work that day happened in a few hours:
  1. `6b45909` 🤖: Detect MakeMKV cell removal and offer a file-mode retry.
  2. `064ff7a` 🤖: Write MakeMKV output to the trace log.
  3. `d139eff` 🤖: Block access file creation while a cell removal is unresolved.
  4. `eddc855` 🤖: **Stop on cell removal** and send the user to MakeMKV's GUI manual mode.
  5. `ad6f6ef` 🤖: **Remove the duration validation** from February.

## Phase 5: Edge-case fixes (Apr–May 2026)

- **7–9 Apr, `84b0d8d` → `d726575` 🤖**: A ddrescue `-b 2048` alignment fix is tried and reverted.
- **9 Apr, `8ee069e` → `a5fb56e` → `3a78f92` 🤖**: The anamorphic aspect-ratio fix took three attempts in about 40 minutes. The final version reads the container SAR from ffmpeg's stream info and passes `-aspect` to the encode.
- **26 May, `69e0995`**: The infinite loop on end-of-input at the menu is fixed. This is the last change to the script. It has no Claude trailer, so git alone can't show whether Claude was involved.

## After that

- **14 Jul 2026**: Doc files are renamed through the web UI (numbered, then un-numbered). The script hasn't changed since.

---

## What changed with Claude Code

- **Bigger commits:** Before Claude, script changes were mostly small edits of 1 to 30 lines, apart from the October 2025 rewrite. Afterwards, commits were often 100 lines or more and added whole features: production rewrites, validation subsystems, removing variants.
- **How the work went:** The Claude-era history shows quick try-and-revise rounds, like four steps of cell-removal design in one afternoon and three aspect-ratio attempts in under an hour. Reverts are kept as their own commits.
- **Documentation:** Every long guide (troubleshooting, imaging basics, Sony authoring, the README rewrite) came out of Claude sessions. The exception is `DVD-VR-RECOVERY.md`, uploaded through the web UI in April 2026.
- **Hand edits continued alongside:** Small tweaks like the `--minlength` change, ISO suffix, renames and README wording kept being made in the GitHub web UI throughout.

## How the script changed overall

- **Imaging:** `dd`, then `ddrescue`, then `ddrescue` with a size margin and `caffeinate`.
- **Scope:** One script, then a family of variants (blend, bob, ProRes mezzanine), then back to one script.
- **Validation:** At first nothing checked the output. A duration check was added in February 2026. In March it was replaced by detecting MakeMKV cell removal from the log and stopping so a person can resolve it.
- **Robustness:** Error handling around each tool call, dependency checks, safer input handling, and fixes for aspect ratio and end-of-input bugs.
