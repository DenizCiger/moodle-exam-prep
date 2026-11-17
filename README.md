# moodle-exam-prep

A personal exam tutor for Claude Code that works from your own Moodle courses.

It downloads what your teacher posted (files, assignment descriptions, starter projects, links, feedback), works out what you already understand and what you don't, and teaches the gaps step by step. You do the practice work yourself, so you can do it alone in the exam. Everything it builds is yours: your diagnosis, your study plan, your drills, your notes. Two people using the same course end up with different setups.

| Part | What it does |
|---|---|
| `exam-prep` skill | The tutor: sets up a study folder, diagnoses you, teaches concept by concept (why → example → teacher's material → explain it back → practice), runs mock exams, remembers progress between sessions. |
| Guard hook (`exam-guard`) | Stops Claude from writing your practice work for you once a drill is handed to you. |
| Moodle access | Uses the [moodle-mcp](https://github.com/DenizCiger/moodle-mcp) server, which the plugin starts for you. |

> Setting this up with an AI agent? Give it this repository and tell it to follow [AGENT_SETUP.md](AGENT_SETUP.md). You'll still have to do step 2 yourself.

## Setup (about 5 minutes)

### 1. Install the two programs
Download the archive for your system from each latest release, check it against its `.sha256`, unpack it, and put the program (`.exe` on Windows) in a folder on your `PATH`:
- `moodle-mcp`: [releases](https://github.com/DenizCiger/moodle-mcp/releases/latest)
- `exam-guard`: [releases](https://github.com/DenizCiger/moodle-exam-prep/releases/latest)

Or build both (needs [Rust](https://rustup.rs)):
```sh
cargo install --git https://github.com/DenizCiger/moodle-mcp
cargo install --git https://github.com/DenizCiger/moodle-exam-prep
```

Check: `moodle-mcp --version` and `exam-guard --version`.

Optional: `pdftotext` (poppler: `brew install poppler`, `sudo apt install poppler-utils`, `scoop install poppler`) for PDF text. Without it, Claude reads PDFs directly, which also works.

### 2. Sign in to Moodle (you, in a terminal)
```sh
moodle-mcp login https://edufs.edu.htl-leonding.ac.at/moodle
```
Enter your Moodle username and password. The password goes to Moodle only and is **not saved**; only the access token Moodle gives back is kept, in your user config folder. Check with `moodle-mcp status`.

Never type your password into a chat with Claude or another agent. The login command is the only place it goes.

### 3. Install the plugin in Claude Code
```
/plugin install moodle-exam-prep --marketplace DenizCiger/moodle-exam-prep
```
(or `/plugin marketplace add DenizCiger/moodle-exam-prep`, then `/plugin install moodle-exam-prep@moodle-exam-prep`). Restart Claude Code afterwards.

## Using it

**First session.** Make an empty folder for the exam (e.g. `~/school/exam-prep/pose-2026-12`), open Claude Code in it, and say what you need, e.g.:

> Prepare me for the POSE exam on 10 December. Course: https://edufs.edu.htl-leonding.ac.at/moodle/course/view.php?id=5154

Claude asks a few questions (exam format, what's covered, which assignments you did yourself or skipped), syncs the course, checks what you already know, and starts teaching. Be honest about skipped or AI-written assignments; that's exactly what it's for.

**What ends up in the folder**

| | |
|---|---|
| `STUDY.md` | your topics with status, plan to the exam, session log, homework |
| `lessons/` | a short lesson per topic, written after you've worked through it |
| `practice/` | your drills. Folders with a `.student` file are yours; Claude only gives hints and reviews there |
| `PLAYBOOK.md` | your exam procedure and checklists, partly in your own words |
| `notes/` | put your own material here: notes, photos of handouts, old tests, grading grids |
| `cards.csv` | flashcards to import into Anki |
| `moodle-data/` | the course mirror (re-syncable, not part of your work) |

**Next sessions.** Open Claude Code in the same folder and say "weiter" (or anything). It reads where you left off, looks at what you practised since, asks a few review questions, then continues. Start a fresh session per study block (1–2 hours) instead of one endless chat. At the end it updates `STUDY.md`, gives you homework, and saves a local git commit.

**Between sessions**: do the drills in your own IDE/editor, and your Anki cards.

**Before the exam**: it runs timed rebuilds and a mock exam. Do those with Claude closed, like the real thing.

Works for other subjects too (writing, calculation, theory, oral exams); it picks the approach from the exam type.

## Privacy and safety
- Moodle access is read-only, and your own submissions aren't downloaded.
- The token from step 2 gives access to your Moodle account like the phone app does. Don't share it. `moodle-mcp logout` deletes it; you can also revoke tokens in Moodle under Preferences → Security keys.
- The guard is a safety net against Claude doing your work, not a security boundary.

## Troubleshooting
| Problem | Fix |
|---|---|
| Claude says it's not signed in / login expired | Run `moodle-mcp login …` again, restart Claude Code |
| `command not found` for `moodle-mcp` or `exam-guard` | The program's folder isn't on your `PATH` (cargo installs to `~/.cargo/bin`) |
| Claude edits your drill folder anyway | `exam-guard` isn't on `PATH`; check `exam-guard --version` and restart Claude Code |
| Login fails although the password is right | If your school signs in through another service (SSO), create a token in Moodle → Preferences → Security keys and set `MOODLE_URL` and `MOODLE_TOKEN` instead |
| Tools don't show up in Claude | `/plugin` to check it's enabled; restart Claude Code |

## License
MIT
