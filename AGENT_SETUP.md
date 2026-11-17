# Setup instructions for AI agents

You are setting up **moodle-exam-prep** for the person you're working with. It is a Claude Code plugin (an exam-tutor skill and a guard hook) that needs two small programs on their `PATH`:

- `moodle-mcp`: the Moodle MCP server, from [DenizCiger/moodle-mcp](https://github.com/DenizCiger/moodle-mcp)
- `exam-guard`: the hook that stops Claude from writing the student's practice work, from this repository

Follow the steps in order and verify each one. Keep your messages to the person brief, and finish with the hand-off at the end.

## Hard rules
- **Never ask for, read, type or store the person's Moodle password.** Step 3 is theirs: they run `moodle-mcp login` themselves in an interactive terminal. The command refuses to run without a terminal.
- Don't read or print the token in the moodle-mcp config file. Use `moodle-mcp status` to check the login.
- Don't change anything on Moodle (the tools are read-only anyway).
- Install only from `DenizCiger/moodle-mcp`, `DenizCiger/moodle-exam-prep` and official sources (rustup, the OS package manager).

## 1. Install `moodle-mcp` and `exam-guard`
Detect the OS and architecture. For **each** of the two programs, prefer the prebuilt release:

| Program | Releases |
|---|---|
| `moodle-mcp` | `https://github.com/DenizCiger/moodle-mcp/releases/latest` |
| `exam-guard` | `https://github.com/DenizCiger/moodle-exam-prep/releases/latest` |

- Asset names: `<program>-<target>.tar.gz` (or `.zip` on Windows), with targets `x86_64-unknown-linux-gnu`, `aarch64-unknown-linux-gnu`, `aarch64-apple-darwin`, `x86_64-apple-darwin`, `x86_64-pc-windows-msvc`.
- Verify each archive against its `.sha256` file from the same release.
- Unpack and put the binary (`.exe` on Windows) into a user-level folder on `PATH` (e.g. `~/.local/bin` on Linux/macOS, `%USERPROFILE%\.local\bin` on Windows; add it to `PATH` if needed and tell the person). On macOS, if Gatekeeper blocks it: `xattr -d com.apple.quarantine <path>`.

Fallback when there's no matching asset: install Rust via rustup if missing (ask first), then
```
cargo install --git https://github.com/DenizCiger/moodle-mcp
cargo install --git https://github.com/DenizCiger/moodle-exam-prep
```

Verify from a **new** shell (so `PATH` is right): `moodle-mcp --version` and `exam-guard --version` both print a version.

## 2. Optional: pdftotext
If `pdftotext -v` fails, offer to install poppler (`brew install poppler` / `sudo apt install poppler-utils` / `scoop install poppler`). Skip if they decline; Claude can read PDFs directly.

## 3. Moodle login (the person does this)
Ask them to run this in their own terminal and tell you when it's done:
```
moodle-mcp login https://edufs.edu.htl-leonding.ac.at/moodle
```
(Use their school's Moodle URL if it's a different one; any Moodle page URL works.) Then verify with `moodle-mcp status` and confirm the name shown is theirs.

If login fails although the password is right, their school probably uses SSO: tell them to create a token in Moodle → Preferences → Security keys and set `MOODLE_URL` and `MOODLE_TOKEN` as user environment variables themselves.

## 4. Install the Claude Code plugin
```
claude plugin marketplace add DenizCiger/moodle-exam-prep
claude plugin install moodle-exam-prep@moodle-exam-prep
```
Verify: `claude plugin list` shows `moodle-exam-prep` enabled. The plugin starts the `moodle` MCP server itself, so don't also add it with `claude mcp add`. If they use Claude Code in the desktop app or an IDE, the same install applies there.

Check for conflicts: if a user-level `exam-prep` skill exists in `~/.claude/skills/exam-prep`, a `moodle` MCP server is already configured (`claude mcp list`), or `~/.claude/settings.json` has an older exam-prep guard hook, tell the person and ask whether to remove the old one.

## 5. Hand off
Tell the person, in their language and in a few lines:
1. Restart Claude Code (so it loads the plugin and the login).
2. Make an empty folder per exam (suggest `~/school/exam-prep/<subject>-<YYYY-MM>`) and open Claude Code there.
3. Start with something like: *"Prepare me for the <subject> exam on <date>. Course: <Moodle course URL>"*, and be honest about which assignments they skipped or generated.
4. Next time: open Claude Code in the same folder and say *"weiter"*.
5. If Claude ever says the Moodle login expired: run `moodle-mcp login …` again.

Point them to README.md (section "Using it") for the details.
