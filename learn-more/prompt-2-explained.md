# Prompt 2 · annotated edition

**Source:** [`prompts/prompt-2-deploy.txt`](https://github.com/Kedirdan/POLI3148_2026Spring_GitHubPages/blob/main/prompts/prompt-2-deploy.txt) (raw, copy-paste this)
**Deck:** [POLI 3148 · Week 13 Lab](https://kedirdan.github.io/POLI3148_2026Spring_GitHubPages/)

This document annotates Prompt 2 section by section. Every block quoted below is the **exact text** Copilot receives; the commentary explains why the language is chosen, which risks it defends against, and what happens in each phase.

---

## What Prompt 2 does

Publishes the folder you currently have open in VS Code as a live webpage on GitHub Pages. Creates a public GitHub repository, pushes your files, enables Pages, waits for the build, and writes a deployment log at `GITHUB_PAGES.md`. Runs once per project; updates thereafter are a single natural-language request to Copilot.

---

## Opening directive

> Please publish this VS Code workspace to GitHub Pages. Use the folder I currently have open, don't ask me for a path. Narrate one sentence before each command, show the tail of output, handle errors yourself.

The phrase *"Use the folder I currently have open"* removes a whole category of failure. Earlier drafts of this prompt asked the student to paste a file path; students mistyped paths, or pasted quoted paths, or used Windows-style backslashes. Now Copilot reads `$PWD` from the workspace, no path is ever typed.

---

## Guiding principles

Prompt 2 opens with four principles that constrain every action Copilot takes. These principles matter because Prompt 2 writes into the student's project files; the principles limit how much of the student's work Copilot can touch.

> • Minimal disruption: only touch files that must be changed to make GitHub Pages work. Leave my existing project structure alone.

Copilot has, in past runs, aggressively restructured student projects. This rule anchors it: if a file change is not strictly required for Pages to work, do not make it.

> • Scientific copy: never move or delete non-web files (`.py`, `.ipynb`, `.md`, `.txt`, `.json`, `.csv`, `data/`, `src/`, etc). If the webpage has to be relocated into `docs/`, COPY (do not move) the web files.

A previous design used `find ... -exec mv` to relocate everything into `docs/`, which mutilated mixed-language student projects (Python dashboards with embedded HTML). The new rule says: treat non-web assets as sacred, and when relocation is needed, use `cp`, not `mv`, so the original root files remain as a safety net.

> • Honest errors: if the build does not finish, do not pretend it succeeded. Report the real state.

An earlier version of Phase G printed the live URL regardless of build status, leaving students staring at a 404 while believing the deploy succeeded. The new rule forces a distinction between *"build finished"* and *"build still running"*.

> • Document what you did: create a new file `GITHUB_PAGES.md` at the root, summarising the exact changes. Never touch the student's existing `README.md` or any other file in the project.

The student's own `README.md` is theirs. Prompt 2 writes a separate, named file, `GITHUB_PAGES.md`, so the deployment log is discoverable without colliding with the project's own documentation.

---

## Phase A · Check prerequisites

> 1. `gh --version` and `gh auth status`. If either fails, stop and tell me to run Prompt 1 first.

Prompt 2 refuses to do any work unless Prompt 1 has successfully completed. This is a precondition guard.

> 2. `pwd` to confirm the workspace root.

A transparency check. Copilot narrates the current directory so the student can verify Copilot is working in the right folder.

> 3. If `.git` already exists AND a remote named `origin` is configured AND there are existing commits, stop and ask me how to proceed. Do NOT overwrite an existing repository.

This guard exists because some students open a folder that is already a clone of another project. Without this check, Copilot would attempt to re-initialise the repo and break the existing remote wiring.

---

## Phase B · Locate the webpage

> 4. Decide which structure to use:
>    a) If `docs/index.html` already exists, reuse it. No file moves needed.
>    b) Else if `index.html` exists at the root (but no `docs/index.html`), create `docs/` and **copy** (use `cp`, not `mv`) only these items into it: ...

Three cases are handled explicitly.

- **Case a**, the project already has `docs/` with an `index.html`. Nothing to move. The common case for students who cloned the demo.
- **Case b**, the project has `index.html` at the root but no `docs/`. Copilot creates `docs/` and copies (does not move) web assets into it.
- **Case c**, neither path exists. Copilot stops and tells the student the project has no webpage to publish.

The list of *what to copy* is deliberately narrow:

- `.html`, `.css`, `.js` files at the root.
- `assets/`, `images/`, `img/`, `css/`, `js/`, `fonts/`, `media/` subdirectories.

And the list of *what NOT to copy* is explicit:

- `.py`, `.ipynb`, `.md`, `.txt`, `.json`, `.yml`, `.yaml`, `.csv`
- `data/`, `src/`, `scripts/`, `notebooks/`, any hidden folder

This whitelist / blacklist pairing is the scientific-copy principle made concrete.

---

## Phase C · Configure `.gitignore`

> 5. Build the list of rules every student project should ignore: ...

The curated default list groups rules by origin:

| Category | Examples | Why |
|---|---|---|
| macOS / editor | `.DS_Store`, `Thumbs.db`, `.vscode/`, `.idea/` | Local-only metadata |
| Python | `__pycache__/`, `*.pyc`, `.venv/`, `.ipynb_checkpoints/` | Regenerated locally |
| Node | `node_modules/`, `npm-debug.log*` | Reinstalled from `package.json` |
| Secrets | `.env`, `*.key`, `*.pem` | Must never go public |
| Large data | `data/raw/`, `*.csv.gz` | Too big for GitHub |

> If `.gitignore` does not exist, create it with the full list above.
> If `.gitignore` already exists, APPEND any rules from the list that are missing. Never remove or rewrite rules the student already has.

The append-never-rewrite rule preserves any project-specific rules the student authored. A student who has already thought carefully about `.gitignore` should not have that thought erased.

---

## Phase D · Init git + first commit

> 6. If `.git` does not exist: `git init -b main`.
> 7. OWNER=$(gh api user --jq .login)
> 8. If `git config user.email` is empty:
>         git config user.name "$OWNER"
>         git config user.email "${OWNER}@users.noreply.github.com"

If the student has no git identity set locally, Prompt 2 configures one using the **GitHub no-reply email**. This is deliberate: a student's personal email should not leak into public commit history.

---

## Phase E · Create GitHub repo + push

> 9. REPO=$(basename "$PWD")
> 10. `gh repo create "$REPO" --public --source=. --remote=origin --push` (this stages + commits + pushes in one go via the flag).
>     If step 10 fails because `.git` exists but there's nothing to commit yet, first run `git add . && git commit -m "Initial deploy to GitHub Pages"` and retry.
> 11. If "name already exists": REPO="${REPO}-$(date +%m%d)"; retry step 10.

The repo name is the folder name. Defaulting to `--public` is not about valuing openness; it is a pragmatic choice. Private repos on free accounts cannot enable Pages. A public repo guarantees Pages works, and the student can switch to private later (with student-free Pro).

The name-collision fallback appends `MMDD` (today's month and day), yielding names like `my-project-0423`. Ugly, but unambiguous.

---

## Phase F · Enable Pages from /docs

> 12. `gh api --method POST repos/$OWNER/$REPO/pages -f source[branch]=main -f source[path]=/docs`

This call posts to GitHub's Pages API, instructing the repo to serve from `main` / `/docs`. GitHub then queues a build.

---

## Phase G · Poll build (≤200 s)

> 13. Poll 25 times, 8 seconds apart:
>         for i in 1..25; do
>           S=$(gh api repos/$OWNER/$REPO/pages --jq .status 2>/dev/null)
>           echo "[$i] $S"
>           [ "$S" = "built" ] && break
>           sleep 8
>         done

Twenty-five iterations × 8 seconds = 200 seconds maximum. On success, the loop breaks early. On timeout, the loop ends without success; Phase I (below) reads that state honestly.

---

## Phase H · Write `GITHUB_PAGES.md`

> 14. Create a new file `GITHUB_PAGES.md` at the workspace root. Never touch the student's existing `README.md` or any other file. Fill in actual values for <OWNER>, <REPO>, today's ISO date, and the list of actions you actually took: ...

`GITHUB_PAGES.md` is a **deployment log**, not a project README. It records:

- Where the page lives (repo URL, live URL, Pages source)
- What Prompt 2 changed (list of actions)
- What Prompt 2 did not change (the student's other files, by name)
- How to update the page later (natural-language message to Copilot)
- How to undo (disable Pages, or delete the repo)
- An invitation to ask Copilot questions about any step

> 15. Commit: `git add GITHUB_PAGES.md && git commit -m "docs: add GitHub Pages deployment log" && git push`.

The log is committed as its own commit with a conventional `docs:` prefix. This keeps deployment and content changes separable in `git log`.

---

## Phase I · Report

> 16. Check final build status from step 13.
>     - If `$S = built`, print the live URL and a pointer to GITHUB_PAGES.md.
>     - Otherwise, print that the build is still running, with the same pointer.

The honest-errors principle in practice. If the Pages API still reports anything other than `built`, Copilot says so explicitly. The student knows to wait and refresh, rather than debugging a 404 that will resolve itself.

---

## Making the repo private later

GitHub Pages on a private repo requires GitHub Pro, free for students through the GitHub Student Developer Pack (verify with your `.hku.hk` email).

```bash
gh repo edit --visibility private --accept-visibility-change-consequences
```

Your page at `<username>.github.io/<repo>/` continues to serve: Pages visibility is independent of repo visibility.

---

## Updating the page later

Edit a file. Ask Copilot: *"Commit and push."* That is the entire loop. Do not re-run Prompt 2.

---

## Common failure patterns

| Symptom | Cause | Fix |
|---|---|---|
| 404 at live URL immediately after deploy | Build still running | Wait, then refresh; check the Phase I message |
| 404 that does not go away | Pages misconfigured, or path wrong | Repo `Settings → Pages`, confirm source is `main` `/docs` |
| "name already exists" | Repo name collision | Copilot retries with a `-MMDD` suffix automatically |
| Files I did not expect are in `docs/` | Root has web files with non-standard names | Ask Copilot: "move X out of `docs/`" |
| Expected files missing from `docs/` | Project uses a subfolder name the whitelist does not know | Ask Copilot: "also copy the `<folder>` folder into `docs/`" |

---

## See also

- [Prompt 1 · annotated edition](./prompt-1-explained.md)
- [Clone prompt · annotated edition](./prompt-clone-explained.md)
- [GitHub Pages official docs](https://docs.github.com/en/pages)
