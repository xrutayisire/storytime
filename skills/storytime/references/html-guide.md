# Filling template.html

Mechanical guide. The template (`assets/template.html`) is a finished interactive page
with empty slots; you produce content-only HTML fragments and substitute them in. Never
modify the template's CSS, JS, or structure — the chrome (acts, nav, theme, progress,
diagram renderers, checklist persistence, print) is already built and self-contained.

## Assembly

1. `cp <skill>/assets/template.html <output-path>`
2. Replace each `<!--SLOT:NAME-->` marker with your fragment.
3. A slot with nothing to say becomes an empty string — empty acts hide themselves
   (label included). Do not invent content to fill a slot.

HTML-escape `<`, `>`, `&` inside code excerpts. No inline `style=`, no new classes, no
`<script>` other than the JSON data blocks described below, no external resources.

## Slots

| Slot | Content |
|---|---|
| `TITLE` | `<headline> — storytime` (plain text) |
| `META` | One line: repo · PR link (`<a>`) or ref · branch → base · N files, +A −D · generated YYYY-MM-DD |
| `HEADLINE` | The headline sentence, plain text |
| `SUBHEAD` | One sentence: the pain before + the trick of the fix, plain text |
| `PICTURE` | Act 1: `<p class="lead">` paragraphs + flow/timeline figures |
| `JOURNEY` | Act 2: one or more `<section class="scene">` blocks |
| `REVIEW` | Act 3: intro `<p class="review-intro">`, `<div class="check">` cards, optional reading-order note |
| `FOOTER` | `<p>` paragraphs for the scope footer (attribution line already in template) |

Tiny-tier mapping: `PICTURE` = headline-level paragraph(s) + 1 diagram only if the fix
is about a flow; `JOURNEY` = exactly one compact scene (the fix, with its code toggle);
`REVIEW` = 1–2 check cards. Never leave a slot empty — all three acts always exist.

## Components

### Act 1 lead paragraph
```html
<p class="lead">When you connect Claude or ChatGPT to Prismic, it receives a temporary
entry pass…</p>
```

### Sequence diagram (rendered by the template from JSON)
```html
<figure class="flow"><script type="application/json" class="flow-data">{
  "title": "Before this PR", "tone": "before",
  "actors": ["Claude", "Prismic", "Auth0 (identity)"],
  "steps": [
    {"from": "Claude", "to": "Prismic", "label": "my pass expired — renew it"},
    {"from": "Auth0 (identity)", "to": "Prismic", "label": "never heard of that ID", "kind": "fail"},
    {"from": "Prismic", "to": "Prismic", "label": "looks up the key in the vault"}
  ],
  "caption": "One sentence under the diagram."
}</script></figure>
```
- `actors`: 2–4 short labels; `from`/`to` must match them exactly.
- `kind`: omit for neutral, `"ok"` (green ✓), `"fail"` (red ✗).
- `from === to` renders a self-step chip on that actor's lane.
- `tone`: `"before"` (red title) or `"after"` (green title); omit for neutral.
- 3–6 steps. Labels under ~45 characters — they must not overlap lanes.
- Use a before/after PAIR when the change alters who-talks-to-whom.

### Timeline (rendered by the template from JSON)
```html
<figure class="timeline"><script type="application/json" class="timeline-data">{
  "events": [
    {"when": "DAY 0", "what": "You connect and log in", "sub": "the only login", "kind": "ok"},
    {"when": "90 DAYS IDLE", "what": "Ticket lapses", "sub": "log in again", "kind": "fail"}
  ]
}</script></figure>
```
3–6 events; `kind`: `ok` / `warn` / `fail` / omit. `when` is a short moment label.

### Scene
```html
<section class="scene" id="scene-1">
  <h2><span class="scene-kicker">Day 0</span>You connect, and two new things happen</h2>
  <p>Narrative prose — clean, no file:line citations.</p>
  ...term cards, code toggles...
</section>
```
Sequential ids (`scene-1`, …). Kicker = the moment; the side nav shows "Day 0 · You
connect, and two new things happen".

### Term card (plain words, at point of need)
```html
<aside class="term">
  <p class="term-name">Plain words · Refresh token, “the renewal key”</p>
  <p>One everyday definition, one analogy, then the real word so the reader recognizes
  it in the code.</p>
</aside>
```

### Code toggle (the only place code and file:line appear)
```html
<details class="code"><summary>Show me the code — the ticket swap</summary><div class="code-body">
  <figure class="hunk">
    <figcaption><a href="https://github.com/o/r/pull/N/files">src/auth.ts:283–297</a></figcaption>
    <pre><code><span class="line add">+  const opaqueToken = mintOpaqueToken()</span>
<span class="line del">-  return auth0Tokens</span>
<span class="line ctx">   …</span></code></pre>
  </figure>
  <p>One sentence: what to notice in these lines.</p>
</div></details>
```
- One `<span class="line add|del|ctx">` per line, leading `+`/`-`/spaces included.
- 4–10 lines per hunk; elide with a `ctx` line containing `…`.
- figcaption links to the PR files view when a PR URL is known; plain text otherwise.

### Check card (Act 3)
```html
<div class="check" data-status="warn">
  <div class="check-head"><span class="check-status">needs your judgment</span>
    <span class="check-title">Could someone stay logged in forever?</span></div>
  <p class="check-q">Why it matters</p><p>One sentence of consequence.</p>
  <p class="check-q">What I found</p><p>The answer — what the code does about this.</p>
  <details class="code"><summary>The evidence — the anchor that must never move</summary>
    <div class="code-body">…hunk…</div></details>
  <p class="check-move">Concrete, finite action. "Nothing required." is valid.</p>
</div>
```
- `data-status`: `ok` / `warn` / `ask`. Status chip text: `checked — looks right` /
  `needs your judgment` / `ask the author`.
- The template appends the "Mark as reviewed" checkbox and tracks progress — do not add it.
- Evidence toggle is optional; "Why it matters" may be omitted on `ok` cards.

### Inference styling (inside any prose)
```html
<span class="inference" data-confidence="likely">probably chosen to match the
notification delay</span>
```
Hedged phrasing ("it looks intended to…") is equally acceptable — one or the other.
