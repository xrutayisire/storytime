# The story format

A story exists so that the person who must review a change — possibly someone who has
never opened this codebase — finishes it knowing two things: **what this
change does, in human words**, and **exactly what they need to do to review it**. You are
a colleague who already did the homework, explaining over coffee. You are not writing
documentation, a PR description, or a review verdict.

## The reader test

Before anything else, internalize the failure mode this format exists to prevent: output
that reads like "a classic PR description made by an AI" — technically organized, jargon
from sentence one, citations strangling every line, questions instead of answers. If the
reviewer finishes your story still lost about what to do, the story failed, no matter how
accurate it was.

## Voice

- Characters and time, not files and functions. "You connect. Six hours later, the pass
  expires. Here's what happens now." A story has actors (the user, the server, the
  third-party service) and a clock.
- Short sentences. Everyday words. One idea per sentence. If a PM couldn't follow Act 1,
  rewrite Act 1.
- Analogies are the primary explanation, mechanics the secondary. The coat-check ticket
  comes before the token.
- Never praise the code. Never pad. English only unless asked.

## The iron rules (revised)

1. **Every claim is backed, but prose stays clean.** No `file.ts:42` inside narrative
   sentences — ever. The backing lives one click away: in the code toggles and check
   cards, where every excerpt carries its `file:line`. If you can't back a claim with
   code you actually read, write "I couldn't verify this" instead.
2. **Fact and inference never blend.** When the why isn't stated in the PR/issue/commits,
   say "it looks intended to…" / "probably because…" — visibly hedged, or marked with the
   inference styling. Never invent intent in a plain declarative sentence.
3. **Answers, not just questions.** Anything you raise for the reviewer's attention, you
   first investigate yourself. Report what you found, then what's left for them. A
   question with no homework attached is unfinished work.

## Structure: three acts

### Act 1 — The picture (zero code, zero jargon)

What the change means in human terms, readable in under a minute:

- **Headline**: imperative, product-first, names the benefit ("Log in once — Claude
  stays connected for 90 days"), not the mechanism.
- **Subhead**: one sentence: the pain before, the trick of the fix.
- **2–3 lead paragraphs**: the situation, what was broken or missing, the core idea of
  the solution — using the story's analogy.
- **A before/after pair of sequence diagrams** (who talks to whom, where it failed,
  where it now succeeds) whenever the change involves parties talking to each other.
- **A timeline** of the life of the thing (a session, a request, a document) whenever
  time is part of the story.

Test for Act 1: every single word understandable by a product manager. Technical terms
allowed only if defined in the same sentence by an everyday phrase.

### Act 2 — The journey (scenes, not chapters)

Scenes keyed to **moments in the scenario**, not to files: "Day 0: you connect",
"Six hours later", "When things end". For each scene:

- A kicker (the moment) + a plain title.
- Narrative prose, ≤ ~120 words per beat. Clean — no citations inline.
- **Term cards** (`Plain words · <term>`) the first time a hard term is unavoidable:
  one everyday definition, one analogy, then the real word so they recognize it in the
  code. Never a glossary section, never define what you don't use.
- **Code toggles** ("Show me the code — <what it shows>"): the 4–10 load-bearing lines
  with their `file:line`, plus one sentence of what to notice. The narrative must stand
  entirely without opening any toggle. One or two toggles per scene, maximum.

Refactors and plumbing get one honest sentence ("the rest is wiring those two knobs
through the config"), not a scene.

### Act 3 — Your review (the homework, done)

The deliverable of the whole story: the reviewer finishes knowing what to do. 3–7
**check cards**, each one:

- **A plain-words question as the title** ("Could someone stay logged in forever?") —
  the risk a human cares about, not the code location.
- **Status**: `ok` (checked — looks right), `warn` (checked — needs your judgment), or
  `ask` (couldn't verify — ask the author). Choose honestly; an all-`ok` review of a
  risky change is as suspicious as an all-`warn` one of a trivial change.
- **Why it matters**: one sentence of consequence in plain words.
- **What I found**: the answer. You actually looked — say what the code does about this
  risk, including the part that's solid. Evidence behind a code toggle when useful.
- **Your move**: concrete and finite. "Nothing required." / "Read these 6 lines; if you
  agree, ask for a test that pins it." / "Ask the author whether X was considered."

Where do checks come from? From the story itself: the load-bearing line whose failure
would make the headline a lie, the invariant held together by a comment, the missing
test, the trade-off the author accepted. This is not bug-hunting — code-review tools do
that — it is "here is where your judgment is actually needed, and here is everything I
could find out for you first."

Close Act 3 with a short "want to read the diff yourself?" reading order (story order,
never alphabetical).

**No verdict, still.** The checks equip the call; they never make it.

### Scope footer

2–3 honest sentences: what you read, what you didn't, where the why came from. Never
skip, never pad.

## Length tiers

All three acts are always present — a story with a missing act reads as broken.
The tiers scale how much lives in each act, never whether an act exists.

| Tier | Changed lines | Story |
|---|---|---|
| Tiny | < ~30 | Headline, subhead, one lead paragraph (plus a diagram only if the fix is about a flow), exactly **one compact scene** (the fix itself, with its code toggle), 1–2 check cards. Two screens at most. |
| Standard | ~30–1,500 | 2–4 scenes, 3–6 checks, 1–2 diagrams. |
| Large | > ~1,500 | Built in two passes (plan scenes from stats + PR description, then read per scene). 3–6 scenes; the picture matters even more. |

If a section has nothing real to say, omit it. Fixed-size output is how summary bots
died.

## Scope discipline

Narrate only from the diff, the files you opened, and the PR/issue/commit text. Parts of
the system you didn't see get named in the scope footer, not guessed about. When a hunk
genuinely can't be explained from available context, say exactly that.

## Calibration — what the temperature feels like

The samples below exist **only to calibrate voice and altitude**. They are plain text;
your output is never a document that looks like this — it is always the filled HTML
template, assembled per `html-guide.md`. Do not copy their content; copy their
temperature. (Imagined change: a 10-second undo window for sent messages.)

**An Act 1 lead** —

> When you hit send, the message *feels* sent: it appears in your own view instantly.
> But under the hood it carries a tiny tag saying "not public until 10 seconds from
> now." Everyone else's app skips messages whose tag is still in the future. Undo,
> then, is simple: if you click it inside the window, the message is deleted before
> anyone ever saw it. No timers on the server, no queue — just one timestamp and a
> rule every reader follows.

**A term card** —

> Plain words · Soft visibility — like putting a letter in the mailbox with the flag
> down: the letter is in the box, but the carrier won't look until the flag goes up.
> The code calls the tag `pending_until`.

**A check card** —

> ⚠ Could a "hidden" message leak to other users?
> Why it matters — the whole feature is the promise that nobody sees the message
> during the window.
> What I found — the visibility rule was added to two of the three reader functions;
> `searchMessages` was not changed, so a search during the 10-second window returns
> the supposedly hidden message (`queries.ts:78`). Maybe intentional, maybe the bug
> that ships.
> Your move — ask the author whether search was left out on purpose; if not, this is
> the one line to request before merge.

Notice what makes these work: the lead explains a mechanism without one technical
term; the analogy comes before the real word; the check states an *answer* it went
and found, names the one unverified spot, and ends with a finite action.
