# Agent Git and PR Workflow

Read this when doing commit, push, PR, code review, or fix reporting.

---

## Commits

### Shortcuts

| Command | Action |
|---------|--------|
| `commit` / `commit quick` | QUICK commit (no push) |
| `commit safe` | SAFE commit (no push) |
| `push` / `push quick` / `commit push` | QUICK commit + push |
| `push safe` | SAFE commit + push |
| `push only` | Push existing commits (no new commit) |
| Any other commit/push request | SAFE |

### QUICK

Allowed if all are true:
- not on `main` or `master`
- no `--force` needed

Steps:

    git add -A
    git commit -m "<type>: <outcome>"
    git push  # if push requested

No approval needed. Stage everything including new files.

### SAFE

1. Inspect scope

       git status --porcelain
       git diff --stat

2. Wait for approval if anything looks ambiguous

3. Stage changes
   - all task changes: `git add -A`
   - selective: `git add -p` or `git add <file>`
   - exclude unrelated changes

4. Commit message format
   - `<type>: <outcome>`
   - type: `feat` | `fix` | `refactor` | `docs` | `chore` | `test`

5. Push if requested
   - never `--force`

6. Report back
   - branch, commit hash, pushed (yes/no)

---

## PR template

Create `.github/pull_request_template.md` with this content:

    ## Summary
    - 

    ## How to test
    - 

    ## Notes
    - Agent review loop: PR Conversation comments only. No inline comments, no review submissions. See AGENTS.md.

Commit and merge to the default branch. New PRs will be prefilled with the template.

---

## PR creation (implementer)

- Use the PR template
- Fill at least Summary and How to test
- Keep the Notes line about agent review loop in the PR body

---

## Code review policy

### Review delivery (strict for agents)

Agents MUST only post PR Conversation comments (top-level).

Agents MUST NOT create:
- inline PR review comments (file/line)
- GitHub review submissions (including "Submit review")

Reason: inline comments and review submissions are commonly missed by tooling. We optimize for deterministic retrieval.

Note: external bots may still post inline comments. Implementer must check them anyway.

### Reviewer format (one comment per round)

Post exactly one top-level PR Conversation comment:

    ### Review - Actionable Findings

    **Blocking**
    - path/file.ext:L10-L15 (symbol): issue → fix → verify

    **Optional**
    - path/file.ext:L42 (symbol): issue → fix

Rules:
- Blocking items must include a verify step (runnable command or objectively checkable)
- Use file:line plus an anchor (symbol name or short unique snippet)
- Keep items actionable, avoid prose

---

## Reading feedback (implementer)

You MUST check all three sources:

1. **Conversation comments** (canonical)
2. **Inline comments** (external bots often use these)
3. **Reviews** (state + body if present)

### Commands

    # Get PR number
    PR=$(gh pr view --json number -q .number)

    # 1. Conversation comments (top-level)
    gh api --paginate "repos/{owner}/{repo}/issues/$PR/comments"

    # 2. Inline comments (file/line)
    gh api --paginate "repos/{owner}/{repo}/pulls/$PR/comments"

    # 3. Reviews (state + body)
    gh api --paginate "repos/{owner}/{repo}/pulls/$PR/reviews"

### Compact output (optional)

    # Conversation comments
    gh api --paginate "repos/{owner}/{repo}/issues/$PR/comments" \
      --jq '.[] | "[\(.user.login)] \(.body | split("\n")[0])"'

    # Inline comments
    gh api --paginate "repos/{owner}/{repo}/pulls/$PR/comments" \
      --jq '.[] | "\(.path):\(.line // .original_line): \(.body | split("\n")[0])"'

    # Reviews
    gh api --paginate "repos/{owner}/{repo}/pulls/$PR/reviews" \
      --jq '.[] | "\(.state) [\(.user.login)] \(.body | split("\n")[0])"'

### If blocking feedback exists only in inline comments

- Fix it anyway
- Include it in Fix Report
- Request reviewer to mirror it into a top-level comment

---

## Fix reporting (implementer)

After addressing findings, post exactly one top-level PR Conversation comment:

    ### Fix Report

    - <item>: FIXED @ <hash> — verified: <command/output>
    - <item>: WONTFIX — reason: <reason>
    - <item>: DEFERRED — tracking: <link>
    - <item>: QUESTION — <question>

### Statuses

| Status   | Meaning                                   |
|----------|-------------------------------------------|
| FIXED    | Change committed, verification passed     |
| WONTFIX  | Intentionally not changing, reason given  |
| DEFERRED | Valid issue, tracked separately           |
| QUESTION | Clarification needed before proceeding    |

---

## Re-review loop

After posting Fix Report:

1. Request re-review explicitly:

       @<reviewer> please re-review. See "Fix Report".

2. If items were unclear:
   - mark as QUESTION in Fix Report
   - ask the exact question needed to proceed

3. If inline-only blocking feedback existed:
   - state in Fix Report that it was fixed
   - ask reviewer to mirror it to top-level for future determinism

4. If external bot created inline threads (optional):
   - reply to inline thread with short acknowledgement + Fix Report reference
   - do not start new inline threads yourself