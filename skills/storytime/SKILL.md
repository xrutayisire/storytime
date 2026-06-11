---
name: storytime
description: >
  Tell the story of a code change. Turns a PR, branch, commit range, or working
  tree into a three-act story: the picture in human words with before/after
  diagrams, the journey scene by scene with code behind toggles, and a review
  checklist where the homework is already done — what to check, what was found,
  what your move is. Produces an interactive HTML page opened in the browser.
  Use when asked to explain a PR, walk through a
  change, tell the story of a diff, or help someone understand or review a
  change. NOT for bug-hunting or merge verdicts — use a code-review skill for
  that.
---

# storytime — the story of a change

You are about to narrate a code change so that the person reviewing it —
possibly someone new to this codebase — finishes knowing **what the change
does in human words** and **exactly what they need to do to review it**.
You are a colleague who did the homework, not a documentation generator. The
output must never read like an AI PR description.

All paths below are relative to this skill's directory.

## 1. Parse the arguments

`$ARGUMENTS` may contain, in any order:

| Argument | Meaning |
|---|---|
| a PR number or GitHub PR URL | narrate that pull request |
| a branch name | narrate branch vs merge-base with the default branch |
| `a..b` | narrate that commit range |
| `--out <path>` | write the story to this path instead of the default |

With no changeset argument: a dirty working tree → narrate it; else the current
branch is ahead of the default branch → narrate that; else narrate the last
commit. Say which changeset you chose in the final recap.

## 2. Gather the changeset

Use whatever your environment provides — there is no required script. You need
three things: **the diff**, **the stats** (files, lines changed), and **the
intent** (PR description, linked issues, commit messages). Typical ways:

- **PR**: `gh pr view <ref> --json title,body,url,baseRefName,additions,deletions,closingIssuesReferences`
  and `gh pr diff <ref>` (add `-R owner/repo` for a URL outside the current
  repo). No `gh` or not authenticated? Fall back to local git against the base
  branch — and say in the scope footer that the PR description was unavailable.
- **Branch**: `git diff $(git merge-base <default-branch> <branch>) <branch>`
  (`--stat` first, then the diff).
- **Range**: `git diff a..b`. **Working tree**: `git diff HEAD` plus
  `git ls-files --others --exclude-standard` for untracked files (read those
  directly).

Pick the tier from total changed lines before reading further:

| Tier | Changed lines | How to read |
|---|---|---|
| tiny | < ~30 | The whole diff, plus enclosing functions of touched code. |
| standard | ~30–1,500 | The whole diff; open every load-bearing file for context. |
| large | > ~1,500 | Two passes: plan scenes from the stat list and PR description alone, then read the diff one scene's files at a time (`git diff … -- <paths>`). |

## 3. Read the format contract

Read `references/story-format.md` now — it is the contract for everything you
write, including the reader test it opens with and the calibration samples it
ends with.

## 4. Understand at the right depth

For every load-bearing hunk, open the touched file and read the enclosing
function — and the caller when cheap to find. Read every intent source that
exists. If you didn't read it, you can't claim it.

## 5. Do the homework

Before drafting, list the 3–7 places where the reviewer's judgment is genuinely
needed: the line whose failure makes the headline a lie, the invariant held by a
comment, the missing test, the accepted trade-off. For each one, **investigate
it now** — read the code, find the answer, decide its status (`ok` / `warn` /
`ask`). These become Act 3's check cards: question in plain words, why it
matters, what you found, the reviewer's move. Never ship a question you didn't
attempt to answer.

## 6. Draft the story

Draft the three acts per the format contract — picture (human words, diagrams,
timeline), journey (scenes by moment, terms at point of need, code behind
toggles), review (the check cards from step 5), scope footer — sized to the
tier.

## 7. Render

Read `references/html-guide.md`, then copy `assets/template.html` to the
output path and replace each `<!--SLOT:*-->` marker exactly as the guide
specifies. Diagrams and timelines are JSON blocks — the template renders them.

## 8. Write, open, recap

- Default output path: `${TMPDIR:-/tmp}/storytime/<repo>-<pr-or-ref>.html`.
  Honor `--out` when given.
- Open it: `open <path>` on macOS, `xdg-open <path>` on Linux. If that fails
  (headless), print the path — not an error.
- Close with a recap of five lines at most: changeset narrated, the headline,
  scene/check counts, the output path. Do not retell the story in the terminal.

## Edge cases

- **Empty changeset**: say so and stop; never narrate nothing.
- **Generated/lockfiles**: one sentence acknowledging them; never narrated.
- **Opaque hunks**: "this part I cannot explain from the available context" —
  honesty over invented coherence.
- **The user asks for a verdict**: the story equips the call, it doesn't make
  it. Point to the check cards.
