---
name: push-firstexample-submodule
description: Commit and push changes in the FirstExample repository to git@github.com:ShengNW/SnwHist_FirstExample.git over SSH, then stage, commit, and push only the FirstExample submodule pointer update in the parent SnwHist repository. Use when the user asks to "push FirstExample", "更新本仓库并更新父仓库指针", or any equivalent publish request for this directory.
---

# Push FirstExample Submodule

## Overview

Use this skill to publish this submodule safely in two steps:
1. Push `FirstExample` itself to `git@github.com:ShengNW/SnwHist_FirstExample.git`.
2. Push only the `FirstExample` submodule pointer change in parent repo `SnwHist`.

## Required Inputs

- Submodule directory (default: current directory)
- Parent directory (default: parent of submodule directory)
- Branch (default: `main` for both repos)
- Commit messages for submodule changes and parent pointer update

## Fast Path (Script)

Run:

```bash
.agents/skills/push-firstexample-submodule/scripts/push_firstexample_and_parent.sh \
  --submodule-dir /absolute/path/to/FirstExample \
  --parent-dir /absolute/path/to/SnwHist \
  --submodule-commit-msg "docs: update FirstExample notes" \
  --parent-commit-msg "chore: update FirstExample submodule pointer"
```

Script behavior:
- Verify submodule origin URL is exactly `git@github.com:ShengNW/SnwHist_FirstExample.git`
- Commit and push submodule changes when present
- Stage only `FirstExample` in parent repo, then commit and push pointer update when changed
- Fail fast if parent repo already has staged files to avoid accidental mixed commits

## Manual Workflow

1. In submodule repository:
   - Run `git status --short --branch`
   - Confirm `git remote get-url origin` is `git@github.com:ShengNW/SnwHist_FirstExample.git`
   - Run `git add -A`
   - Run `git commit -m "<submodule message>"` (skip if no content changes)
   - Run `git push origin main`
2. In parent repository:
   - Run `git add FirstExample`
   - Run `git diff --cached --name-only` and ensure only `FirstExample` is staged
   - Run `git commit -m "<parent pointer message>"` (skip if pointer unchanged)
   - Run `git push origin main`
3. Report:
   - Submodule commit SHA
   - Parent commit SHA (if pointer changed)
   - Final `git status --short --branch` for both repos

## Guardrails

- Always push with SSH remotes.
- Keep parent commit scoped to the `FirstExample` submodule pointer only.
- Avoid destructive git commands (`reset --hard`, checkout discard, etc.) unless explicitly requested.
- Confirm current prompt is appended to `PromptHist.md` before final submodule commit.
