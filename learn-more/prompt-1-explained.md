# Prompt 1 · annotated edition

**Source:** [`prompts/prompt-1-setup.txt`](https://github.com/Kedirdan/POLI3148_2026Spring_GitHubPages/blob/main/prompts/prompt-1-setup.txt) (raw, copy-paste this)
**Deck:** [POLI 3148 · Week 13 Lab](https://kedirdan.github.io/POLI3148_2026Spring_GitHubPages/)

This document annotates Prompt 1 section by section. Every block quoted below is the **exact text** Copilot receives; the commentary underneath explains why each line is written the way it is.

---

## What Prompt 1 does

Installs GitHub's command-line tool, `gh`, on your computer and logs you into your GitHub account. You run this **once per computer, forever**. After it succeeds, Copilot can push code to GitHub on your behalf.

---

## Opening directive

> Prepare my machine so I can use GitHub from the terminal. Run every step below in my integrated terminal. Narrate one sentence before each command, show the tail of its output, and handle errors yourself. Stop only if truly blocked.

Four constraints are packed into this opening sentence.

- *"Prepare my machine..."* gives Copilot a concrete goal, not a vague task.
- *"Run every step below in my integrated terminal"* confines execution to the VS Code terminal, preventing Copilot from spawning subprocesses it cannot observe.
- *"Narrate one sentence before each command, show the tail of its output"* makes the flow human-readable. Students watch what is happening, they do not face a wall of cryptic output.
- *"Handle errors yourself. Stop only if truly blocked"* grants autonomy to retry or recover, while reserving the right to halt when something is genuinely wrong.

---

## Global rules

> • Do NOT use `exit`, `||`, or nested subshells inside polling loops, those can close the wrapper shell. Use plain for-loops with `break`.

Copilot runs each terminal command inside a wrapper shell it controls. Certain shell constructs, `exit`, logical-or short-circuits, and nested subshells, can accidentally terminate that wrapper, which kills the entire session. The rule forces explicit, safe loop constructs.

> • When a command prompts for the macOS admin password, POST this message clearly in the chat: WARNING: macOS is asking for your admin password in the terminal below...

macOS asks for your login password when installing software, but the password field does not echo characters as you type. Students who have not seen this before assume the input is broken. Copilot is told to post a clear, boxed instruction in chat so students know what to expect.

> • When gh prints a one-time device code, POST it prominently in chat (not buried), with 4-step browser instructions and a 15-minute expiry warning.

The device-code login produces an 8-character code buried in terminal output. If Copilot does not surface it explicitly, students miss it, and the code expires after 15 minutes. This rule makes the code unmissable.

> • If any install step fails, DO NOT silently continue. Stop, clearly explain the failure, and point me to the manual download URL.

Automatic installers can fail for many reasons: no Homebrew on macOS, no `winget` on Windows, a flaky network. Without this rule, Copilot might proceed silently and leave the student with a broken install and misleading success messages. Honest failure is both a technical and a pedagogical value.

---

## Phase A · Install `gh`

### Steps 1–2, preflight

> 1. Run `gh --version`. If it succeeds, skip to PHASE B.
> 2. Detect my OS (run `uname -s` on Unix or check `$env:OS` / `ver` on Windows).

Step 1 makes Prompt 1 idempotent: running it twice on a set-up machine is a no-op. Step 2 branches the install logic by operating system.

### Step 3, macOS

> 3a. Run `which brew`. If Homebrew is present, run `brew install gh`, then go to step 6.

If Homebrew exists, use it. Homebrew is the standard macOS package manager, and students who have it have already consented to its conventions.

> 3b. Otherwise do NOT install Homebrew (too invasive). Try the signed .pkg install: ...

This restraint is deliberate. Installing Homebrew on a student's computer is invasive: it writes to system paths, modifies shell configuration, and is a long operation. Instead, the prompt downloads Apple's signed `.pkg` installer directly from GitHub's releases. The `sudo installer` command that follows writes only to `/usr/local/bin` and is scoped to `gh` alone.

> 3c. Manual-install message for macOS: ... download the file ending in `_macOS_arm64.pkg` (Apple Silicon) or `_macOS_amd64.pkg` (Intel).

If the automatic path fails, Copilot hands control to the student with a clear manual route. The student does not need to understand the recovery logic; they open a link and double-click.

### Step 4, Windows

> 4a. Try `winget install --id GitHub.cli -e --silent...`
> 4b. If winget is unavailable OR the command fails, STOP and print the manual-install message.

`winget` ships with recent Windows 10 and 11. Older machines lack it; for those, the fallback is a classic `.msi` double-click.

### Step 5, Linux

> 5a. Follow https://github.com/cli/cli/blob/trunk/docs/install_linux.md for your distribution.

Linux distributions diverge too widely to automate safely. Prompt 1 defers to the official per-distro guide rather than guessing.

### Step 6, confirm

> 6. Confirm with `gh --version`. If it still fails, STOP.

A final sanity check. If `gh --version` still fails, Copilot halts and reports, preventing Phase B from running against a broken install.

---

## Phase B · Authenticate

### Step 7, preflight

> 7. Run `gh auth status`. If already logged in, skip to step 10.

Idempotent. Students who have logged in previously skip straight to the final verification.

### Step 8, issue the login request

> 8. Run: `gh auth login --hostname github.com --git-protocol https --web --scopes repo,workflow,delete_repo`

Four flags, each deliberate.

- `--hostname github.com`, targets public GitHub (not Enterprise).
- `--git-protocol https`, uses HTTPS for pushes. SSH keys would require separate setup.
- `--web`, triggers the browser-based device flow, the simplest login path in a classroom.
- `--scopes repo,workflow,delete_repo`, requests three permissions:
  - `repo`: read and write your own repositories (required)
  - `workflow`: modify GitHub Actions files (unused today, kept for future labs)
  - `delete_repo`: delete your own repositories (optional, makes cleanup easier)

### Step 9, surface the device code

> 9. Extract the 8-digit device code from output. Post this boxed message in chat: ...

This step enacts the global rule about surfacing the device code. The boxed layout, the prominent code, and the explicit 15-minute expiry warning prevent the two most common failures: students missing the code, and students taking too long in the browser.

> Poll with a safe for-loop (no exit, no ||):
> for i in 1..20; do gh auth status && break; sleep 5; done

Twenty iterations at 5 seconds each, 100 seconds total. `&&` triggers `break` on success. Note the absence of `||` and `exit`, honouring the global rule: the wrapper shell stays alive.

### Step 10, final confirmation

> 10. Run `gh api user --jq .login` to confirm. ... print: Environment ready. Logged in as <username>.

The final API call is an end-to-end check: not only is the local auth token valid, but the GitHub API accepts it and returns your login. This is the strongest single-command verification.

---

## How to verify Prompt 1 worked

Open a VS Code terminal and run:

```bash
gh --version
gh auth status
gh api user --jq .login
```

All three should succeed and show your GitHub username.

---

## Common failure patterns

| Symptom | Cause | Fix |
|---|---|---|
| `gh: command not found` after install | New `PATH` not loaded in current shell | Close and reopen the VS Code terminal, or restart VS Code |
| Device code expired | Took longer than 15 minutes in browser | Re-paste Prompt 1; Copilot issues a new code |
| `sudo` password does not echo | macOS does not echo password characters, by design | Type carefully, press Enter |
| Homebrew present but `brew install gh` fails | Stale Homebrew tap | Run `brew update`, then retry |
| Neither automatic path works | GitHub release structure changed, or network restricted | Manual download: https://github.com/cli/cli/releases/latest |

---

## See also

- [Prompt 2 · annotated edition](./prompt-2-explained.md)
- [Clone prompt · annotated edition](./prompt-clone-explained.md)
- [GitHub Pages official docs](https://docs.github.com/en/pages)
