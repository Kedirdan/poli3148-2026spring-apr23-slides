# Clone prompt · annotated edition

**Source:** [`prompts/prompt-clone.txt`](https://github.com/Kedirdan/POLI3148_2026Spring_GitHubPages/blob/main/prompts/prompt-clone.txt) (raw, copy-paste this)
**Deck:** [POLI 3148 · Week 13 Lab](https://kedirdan.github.io/POLI3148_2026Spring_GitHubPages/)

This document annotates the Clone prompt section by section. Every block quoted below is the **exact text** Copilot receives; the commentary explains why the prompt exists at all and why each step is the way it is.

---

## What the Clone prompt does

Downloads the instructor's demo project to your Desktop, then detaches it from the instructor's GitHub account, so you can practise the deploy flow on a project that is guaranteed to work. Run this **only** if you do not have your own HTML project ready for today's lab.

---

## Why the Clone prompt exists

Prompt 2 (the deploy prompt) requires a project that contains at least one `.html` file. Students who arrive without such a project, forgot the In-Class Exercise, or finished it late, would otherwise be stuck watching classmates deploy.

The Clone prompt gives them a known-good copy of the instructor's demo, `Kedirdan/POLI3148_2026Spring_GitHubPagesDemo`. Running Prompt 2 on the clone produces a live page within a minute, letting the student focus on learning the deploy flow rather than debugging their own HTML.

---

## Opening directive

> Please clone my instructor's demo project to my Desktop so I can practice the deploy flow on it. Narrate one sentence before each command, show the tail of output, handle errors.

The framing *"so I can practice the deploy flow on it"* sets Copilot's intent: this is a practice clone, not a collaboration or a fork. The student is not contributing back to the instructor's repo; they are starting from a copy to run Prompt 2 against.

---

## Setup

> SOURCE="https://github.com/Kedirdan/POLI3148_2026Spring_GitHubPagesDemo"
> TARGET="~/Desktop/POLI3148_2026Spring_GitHubPagesDemo"

The destination is the Desktop, not an arbitrary path, for two reasons. First, it is consistent across machines, no instructor has to ask *"where did you put it?"*. Second, the Desktop is universally writable without permission issues.

---

## Steps

### Step 1, idempotency check

> 1. `cd ~/Desktop` and check whether `POLI3148_2026Spring_GitHubPagesDemo` already exists. If so, stop and tell me it's already cloned.

If the student has run this prompt before, there is already a folder with the same name on the Desktop. Re-cloning would overwrite their in-progress work. Copilot halts with a clear message instead.

### Step 2, perform the clone

> 2. `git clone "$SOURCE" "$TARGET"`. Show the tail of the clone output.

A standard clone. The tail of the output is visible to the student, so they see the progress indicator and final commit count.

### Step 3, detach from the instructor's remote

> 3. `cd "$TARGET"` and `git remote remove origin` so the clone is detached from my instructor's repo (ignore error if no origin).

**This step is the reason the Clone prompt is not "just `git clone`".**

After `git clone`, the cloned folder has a remote named `origin` pointing to the instructor's GitHub repo. If the student then runs Prompt 2 without removing that remote, Prompt 2's `git push` tries to push to the **instructor's** GitHub repository. The student has no write access there, so the push fails with a permission-denied error.

Removing `origin` severs that link. The cloned folder now has no remote. When Prompt 2 runs, `gh repo create --source=. --remote=origin --push` creates a fresh repo under the **student's** GitHub account and points the local folder's `origin` to it. This is the correct wiring.

The trailing *"ignore error if no origin"* exists because a future version of `git clone` might not set `origin` automatically; the prompt is robust to that.

### Step 4, closing message

> 4. Final message, on its own line:
>    Demo ready at ~/Desktop/POLI3148_2026Spring_GitHubPagesDemo. Open that folder in VS Code (File → Open Folder), then paste Prompt 2.

The message gives the student the next concrete action, not a vague *"you're done"*. They open VS Code, open the folder, paste Prompt 2. No guesswork.

---

## What the cloned folder contains

```
POLI3148_2026Spring_GitHubPagesDemo/
├── dashboard.py         # Python script that generates the HTML dashboard
├── data/                # V-Dem data files
├── docs/
│   └── index.html       # The pre-built webpage, ready to publish
├── README.md
└── requirements.txt
```

Because `docs/index.html` already exists, Prompt 2 takes the fast path: no file copying in Phase B, no `.gitignore` rewriting (one is already present). It creates a repo under the student's account, enables Pages, and publishes. About a minute later, the student has a live URL mirroring the instructor's demo but hosted on their own GitHub account.

---

## After the lab

The student's deployed clone lives at `https://<their-username>.github.io/POLI3148_2026Spring_GitHubPagesDemo/`. It is theirs: they can edit `dashboard.py`, change the subtitle, add sections, push updates. The instructor's original repository is untouched.

To delete the clone later:

```bash
gh repo delete <their-username>/POLI3148_2026Spring_GitHubPagesDemo
```

---

## See also

- [Prompt 1 · annotated edition](./prompt-1-explained.md)
- [Prompt 2 · annotated edition](./prompt-2-explained.md)
- [GitHub Pages official docs](https://docs.github.com/en/pages)
