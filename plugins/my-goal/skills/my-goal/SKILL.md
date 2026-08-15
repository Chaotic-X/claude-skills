---
name: my-goal
description: Use when the user invokes /my-goal, says "turn this into a goal prompt" / "write me a goal prompt", rambles a desired outcome and asks for the prompt that will make it happen in a fresh session, or asks to resume, restart, or abandon a goal prompt they started earlier. The deliverable is one paste-ready goal prompt, never the build itself — do NOT use when they want the thing built right now in this session.
---

# Goal Prompt Writer (/my-goal)

Turn the user's ramble into one exceptional goal prompt for a fresh autonomous session — through a
draft-first walkthrough where every inference is surfaced for their confirm/change. You are not
building the thing, and you are not interviewing them from scratch: you draft, they react.

Throughout this skill, **the owner** means the person named in `~/.claude/my-goal/environment.md`.
Use their actual name in what you write; "the owner" is a placeholder for you, not for them to read.

## Configuration

Ground truth lives at `~/.claude/my-goal/environment.md` — delivery defaults, standing clauses, tool
roster, model routing, cost posture, session retention.

**If that file does not exist, invoke the setup skill first**, let it finish, then continue from
Phase 1 with the ramble the user already gave you. The skill is named `my-goal-setup` when installed
into a skills directory, and `my-goal:my-goal-setup` when installed as a plugin — use whichever
appears in your available-skills list. Do not ask them to repeat themselves, and do
not try to run without a config — a goal prompt that names no real paths or tools is worthless.

In-progress, paused, and abandoned walkthroughs live in `~/.claude/my-goal/sessions/`. Both paths sit
beside `environment.md`, deliberately outside any version-numbered plugin directory, so updating the
plugin never destroys the owner's config or their unfinished goals.

## Process

### Phase 0 — Prune, then offer

Before anything else, scan `~/.claude/my-goal/sessions/`.

**First, prune.** Any file with `status: abandoned` whose `abandoned:` datestamp is older than the
retention window gets deleted — and one line per deletion appended to
`~/.claude/my-goal/sessions/_abandoned-ledger.md` first:

```
2026-09-13 · comfyui-style-sheets · started 2026-08-10, abandoned 2026-08-14 at 4/7 · file deleted
```

The window comes from the **Keep abandoned goals for** line in `environment.md`. If that line is
absent — a config written before the field existed — use **30 days** and say nothing about it. If
anything was pruned, report it in one line; if nothing was, say nothing. Never delete silently, and
never delete without the ledger line landing first.

**Then offer.** Files whose `status:` is neither `delivered` nor `abandoned`:

- **paused** — the owner said `pause`.
- **interrupted** — `in-progress` from a session that ended without delivering (context clear, crash).

If any exist, offer them before starting fresh — one line each: slug, when, `position N/7`, and the
one-line desire from part 1. The owner picks one, says they want a new goal, or abandons one from the
list. If none exist, say nothing about it and go straight to Phase 1.

**Resuming is warm.** Restore the confirmed parts and position, then re-read `environment.md` (it may
have changed) and re-run every check in the file's `## Verified` block. Anything that has vanished,
moved, or been renamed is presented as a **change** on the part that named it — which marks that
part `stale` and ripples normally. Only then continue at `position`, and flip `status` back to
`in-progress` on the first write — a walkthrough actively in progress should never sit on disk marked
`paused`. A path verified last Monday is not a path verified today.

### Phase 1 — Intake
Accept the ramble as-is; voice-to-text tolerant (interpret intent over literal words: "Quad MD"
means CLAUDE.md, "Netlefi" means Netlify). Read `~/.claude/my-goal/environment.md` once. Ask nothing
yet. Open the session file with the ramble stored verbatim — slug it from the deliverable in two or
three hyphenated words (`comfyui-style-sheets`, `rhune-landing-page`), so a list of paused goals is
readable without opening any of them. `position` reads `1` from the moment the file is opened,
before Phase 2 has drafted anything.

### Phase 2 — Draft all 7 parts
Draft every part of the anatomy below immediately, filling gaps with the most probable reading of the
ramble plus environment defaults. Verify before you name: anything you plan to name in the prompt
(path, skill, MCP) gets a 30-second existence check (`ls` the path, check the tool list) — the live
environment is the source of truth, not the roster file. Record each check in the session file's
`## Verified` block with the time. Name only what you verified AND what is load-bearing (usually 2–4
things).

### Phase 3 — Walkthrough (the point of this skill)
Present the draft one part at a time, in order:

> **N/7 — [Part name]**
> "…drafted prose for this part…"
> Inferred: [one line per assumption — and what changes if it's wrong]

…immediately followed by the menu, printed verbatim:

```
────────────────────────────────────────
(y)es · (c)hange <what> · (b)ack [n]
(r)estart · (p)ause · (a)bandon
```

Every fact in the drafted prose that came from neither the ramble nor a verified check gets an
Inferred line — no silent specifics. **Print the menu after every part**, identically, so the owner
never has to remember what is available.

**The six verbs.**

| Verb | Key | Means | You do |
|---|---|---|---|
| `yes` | `y` | accept as drafted | mark `confirmed`, advance |
| `change <what>` | `c` | anything about this part should be different — naming another part routes to it | fix it, mark `confirmed: changed`, **run the ripple pass**, advance |
| `back [n]` | `b` | return to an earlier part | ask which step unless they named one, set it to `awaiting`, rewind position; run the ripple pass after they change it |
| `restart` | `r` | the opening statement itself was wrong | take a new ramble, clear all parts, back to position 1 |
| `pause` | `p` | stop here, coming back | `status: paused`, tell them the resume command, end the turn |
| `abandon` | `a` | stop here, not coming back | confirm first, then `status: abandoned`, end the turn |

**A key opens the action; it does not fire it.** `y` and `p` are the only ones that complete on
their own — a yes has no follow-up question, and a pause has nothing to ask. Every other key asks the
question its verb implies and waits: `c` asks what's wrong, `b` asks which step
they're going back to, `r` asks for the new opening statement, `a` asks them to confirm and names
what will be lost. A single keystroke never destroys work.

**One character is a shortcut; anything longer is prose.** A lone `c` selects change. "change the
resources part" is prose — parse it and act. "can we make it warmer" is prose that *infers* to
change. The one-character rule is the only thing keeping those from colliding, so apply it strictly:
if the reply is longer than one character, read it for meaning, never for its first letter.

**Unmapped keys get a guess, never silence.** A bare `n` is not in the map, but it plainly means
something is wrong — come back with "reading that as *something's wrong* — what is it?" rather than
nothing. Same for any other stray character: name your best reading of it and ask. The owner should
never type something and get no acknowledgement that they typed it.

**Ambiguous prose never resolves to a destructive verb.** "cancel this", "forget it", "never mind"
and their cousins can mean *abandon the goal* or *disregard what I just typed*. When a reading could
be `abandon` and could be something smaller, ask which. Guessing generously is right everywhere else
in this skill and wrong here — this is the one verb that cannot be undone.

**The keywords are optional.** Infer the verb from what they actually say. "No, it's ToT not SoO" is
a change; "make it warmer" is a change; "go back to the resources part" is a back; "forget this
whole thing" is an abandon. The menu names categories, not a syntax they must obey — this skill is
voice-to-text tolerant everywhere else and the menu is no exception.

**Ripple pass** (after any change, or after a back-edit). **Scope it first:** a downstream part
ripples only if it actually depends on what changed — it names the fact, rests on it, or was drafted
from it. Parts with no dependency are skipped without re-drafting. A wording tweak must not cost six
re-drafts. Then, for the parts that do depend on it, re-draft and diff each against the text already
confirmed, and mark only the ones that *materially moved* as `stale`. Re-present the stale parts in
order, labeled with what caused the change. Parts that did not move keep their confirmation and are
never shown again. Never silently revise a part the owner already confirmed. The diff is what
decides — the owner classifies nothing.

**On a first forward pass there is nothing downstream to protect.** Parts ahead of the current one
are unpresented drafts, not commitments. **Confirmation, not presentation, is what earns ceremony** —
a part the owner has seen but not confirmed (they backed away from it, or a change aimed at it was
refused) is still a draft. Re-draft them on the changed premise silently and carry
on to the next part — no diffing, no `stale` markers, no re-presenting. The ripple's ceremony exists
for parts the owner has already confirmed, which in practice means changes reached through
`back`.

**An explicit "only this" suppresses the ripple.** If the owner says "just change this word, don't
touch anything else," they win: patch in place and advance. They are allowed to override the diff —
they are simply never *required* to classify for it. **Record the suppression on that part in the
session file.** A later change that ripples into the same part must preserve the wording the owner
protected, or say plainly that it could not — otherwise "nothing else" holds only until something
else happens to reach it.

**The ripple runs forward; contradictions do not.** A change can invalidate a part *earlier* than the
one being edited — a change to 2 that redefines the deliverable leaves part 1's stakes describing
something else — and the ripple, which only ever flows downstream, will never catch it. So check
upstream by hand: after any change, re-read the confirmed parts behind the cursor and ask whether the
new premise still fits them. If one no longer holds, say so and offer a `back` to it. A goal prompt
whose parts 1 and 2 disagree is worse than one that asked a question.

**`back` always establishes its destination before it moves.** Pressing `b` prints the parts *behind*
the cursor — only those, since parts ahead are unpresented drafts and not destinations — on one line,
in the menu's own idiom:

```
back to: (1) desire · (2) quality bar · (3) resources · (4) freedom
```

No status tags. Everything behind the cursor is confirmed by definition, so they would be ink without
information. A reply that already names a part (`b 2`, "back to the resources part") has answered the
question; take it and go. Never rewind to a part the owner didn't name.

**The candidate picker is not the `back` picker.** `back` moves backward, so its picker offers only
what sits behind the cursor. Disambiguation asks a different question — *which part did you mean* —
and the answer can be the part on screen, which is often the strongest reading. So list the parts
that genuinely answer to the phrasing, wherever they sit, and mark the current one:

```
did you mean: (2) quality bar · (5) verification loop — on screen now
```

**Never print a picker that omits a live candidate.** A picker whose options steer away from the most
likely reading is worse than the guess it replaced: it looks like a question and behaves like an
answer. Parts *ahead* of the cursor stay out either way — they are unpresented drafts, so the owner
cannot have meant one. Nothing has moved and nothing has changed while the picker is on screen; say
so, and stay where you are until they point.

**`back` leaves the parts in between exactly as they are** — confirmed, and still confirmed. Going
5→2 does not un-confirm 3 and 4; they are re-drafted only if the ripple from part 2 actually reaches
them. Don't discard them, don't de-confirm them, and don't re-present them on the way back down. When
the ripple finishes, the cursor returns to the frontier — the furthest part the owner had reached —
not to the part they backed into.

**A lone `c` is scoped to the part on screen; prose that names another part is routed.** The key
itself names nothing, so the part in front of them is the only thing it could mean — a bare `c` never
reaches backward. Prose is different: it can carry a destination, and this skill already reads
anything longer than one character for meaning rather than for its first letter.

**Route only when the naming resolves to exactly one part.** That is what "establishes a destination"
means throughout this skill — one unambiguous referent, not the owner's confirmation. "change the
quality bar" names part 2 and nothing else, so route it. Naming loose enough that two parts could
answer to it is *not* established: print the **candidate picker** (below) and let them point, rather
than committing to your own parse and announcing a guess as though it were a fact. And when the
naming resolves to the part already on screen, there is nothing to route — make the change where you
are, with no announcement about going anywhere.

**Resolution comes first.** The on-screen rule is not a shortcut past the ambiguity test — it applies
only once the naming has resolved. A phrase that could mean the part on screen *or* an earlier one
has not resolved, so it gets the candidate picker, with the on-screen part listed among the
candidates. Deciding it must mean the part in front of you because that is where you happen to be
standing is the same announced guess by a quieter route.

When it does resolve, announce the move in one line before you make it, the same way every time:

> That's 3/7 — going back.

Then apply the change there and ripple forward. **If the prose named a destination but carried no
edit** — "change the resources part", and nothing else — route and announce, then ask what should
change once you are there. Never invent the edit to save a turn.

Afterward the cursor returns to the frontier, the furthest part the owner had reached — but **stale
parts come first.** Anything the ripple invalidated is re-presented and resolved before the frontier
is restored; you cannot be standing at the frontier with a broken part behind you. Announce, never
teleport silently, and never make the owner rephrase something you already understood.

**Fatigue valve:** "rest looks good", "skip to the end", or anything similar batch-confirms
everything remaining — jump straight to Phase 4 with zero further questions.

**Write the session file after every state change** (schema below) — a confirmed part, a `back`, a
restart, a pause, an abandon — before presenting anything else. Open it in Phase 1 with the ramble,
and write it again at the end of Phase 2 with the drafts and the `## Verified` block, so a crash
before the first confirmation still leaves something to resume from. The file is the source of truth
for `back`, `pause`, `restart`, `resume`, and the ripple diff — not your memory of the
conversation.

### Restart

`restart` is for when the **premise** was wrong — not one part, the whole opening statement. It
discards every confirmed part, so when any are confirmed it names the loss first — "3 of 7 parts
confirmed will be cleared — restart?" — the same courtesy `abandon` gets. With nothing confirmed yet
there is nothing to lose: just go. Then ask for the new or revised statement and, in the same session
file:

1. Append the old ramble to a `## Superseded rambles` section with a timestamp. Never delete it — it
   is how a warm resume months later understands what changed and why.
2. Replace `## Ramble` with the new statement, verbatim.
3. Clear every part, reset `position: 1`, and keep the `## Verified` block only where the checks
   still apply to the new premise — re-run anything you are unsure of.
4. Re-slug and rename the file if the deliverable changed. Same goal, new premise, one file.

Then re-draft all 7 from scratch and present 1/7. A restart is not a change and does not ripple —
there is nothing downstream left to ripple into.

### Abandon

`abandon` ends a goal for good. Like `restart` it destroys confirmed work, and unlike `restart` there
is nothing left standing afterward — so it **confirms first**: name what
will be lost — "4 of 7 parts confirmed — abandon it?" — and wait for a yes. Count the actual
confirmed parts; that number is computed every time, never the literal 4. On confirm:

1. Write `status: abandoned` and an `abandoned:` datestamp into the frontmatter.
2. Tell them it will be kept for the retention window and then deleted automatically.
3. End the turn. No summary, no salvage offer, no asking what went wrong.

Abandoned goals are never offered at Phase 0. `a` works anywhere: mid-walkthrough, or on a paused
goal offered in the Phase 0 list.

### Phase 4 — Assemble + deliver
Weave the confirmed parts into flowing first-person prose (the owner speaking to the fresh session),
150–350 words, no headers or bullets inside the prompt. Deliver exactly this, in order:

1. The prompt in one fenced code block.
2. Copy it to the clipboard (`pbcopy` on macOS, `xclip -selection clipboard` or `wl-copy` on Linux —
   tell them it's on the clipboard).
3. One **Launch** line — the recommended command for the fresh session, from the launch guidance in
   `environment.md`.
4. The **Expect** block (below).
5. An **Assumptions** list covering ONLY parts batch-confirmed via the fatigue valve — walked-through
   parts are confirmed facts, not assumptions.

Nothing else. Then mark the session file `status: delivered` and leave it in place; delivered goals
are the owner's record of what they have asked for before, and they never age out.

## The Expect block

Two lines, for the owner only. **It never goes inside the generated prompt** — the fresh session
already carries the burn posture from `environment.md`; this exists so the owner can decide whether
to launch now.

```
**Expect**  ~45–75 min unattended — longest if the verification loop
            restarts on a weak concept rather than patching one.
**Burn**    Moderate — call it a fifth of a weekly window at Max.
            Lever if that's too heavy: keep exploration in subagents.
```

**Time** comes from the shape the walkthrough already established: deliverable count and per-item
work, the minimum three verification passes plus up to ~3 owner-gate loops, whether phases run
sequentially, and whether anything slow (renders, builds, transcription) sits in the loop.

**Burn** comes from agent fan-out and tier, expected main-loop turns, and whether exploration is
offloaded to subagents. Its **unit follows the `Plan` line in the `## Cost posture` block of
`environment.md`**. A config written before that block existed simply won't have one — treat a
missing block as "nothing set" and use the last row:

| Plan on file | Express burn as |
|---|---|
| Max / Pro / Team seat | a fraction of the binding window — "about a fifth of a weekly window" |
| API pay-as-you-go | a dollar band — "roughly $4–9" |
| nothing set | bare `light` / `moderate` / `heavy`, no units, plus a nudge to run the setup skill |

If `Watch level` is missing, treat it as `relaxed`. If it is `watched closely` and the reading is
`heavy`, say plainly that it may not survive
one window and suggest phasing the goal into two. If it is `relaxed`, report the band and move on.

**Bands only, never point numbers**, and each line names its single biggest blow-out factor. This is
a reasoned estimate from the goal's shape, not a measurement — it cannot see how much of the window
is already spent today. Do not imply otherwise.

## Session file

`~/.claude/my-goal/sessions/YYYY-MM-DD-<slug>.md`, rewritten after every confirmed part.

```markdown
---
goal: comfyui-style-sheets
started: 2026-08-10T16:52-06:00
updated: 2026-08-10T17:20-06:00
status: in-progress          # in-progress | paused | delivered | abandoned
position: 5                  # part currently awaiting confirm
abandoned:                   # datestamp, only when status is abandoned
---

## Ramble
<the current ramble, verbatim — never rewritten by a change; replaced only by a restart>

## Superseded rambles
<older rambles a restart replaced, each with the time it was superseded>

## Verified
- /Volumes/x8/Claude_Resource_Dir/Alex/ — exists (16:55)
- ComfyUI MCP xSSD4:8188 — reachable (16:55)

## Parts

### 2 — Quality bar  [confirmed: changed]
<drafted prose>
Inferred:
- style-locked across the set — CONFIRMED
Change: "ToT not SoO" -> premise fixed, rippled to 5

### 5 — Verification loop  [awaiting]
<drafted prose>
```

Per-part status is one of `awaiting`, `confirmed`, `confirmed: changed`, `stale`.

**Reading older files.** Sessions written before v1.2.0 may carry `confirmed: adjusted` (from when
`adjust` was a separate verb) or `confirmed: corrected` (from when `change` was called `correct`).
Read either as `confirmed` and carry on — do not rewrite the marker, and do not treat it as an error.
Those files resume normally.

## The 7-part anatomy

1. **Desire + stakes** — concrete deliverable + quantity + why it matters, with real numbers/context
   when they exist.
2. **Quality bar** — one vivid sentence of what excellent looks like, plus the hard non-negotiables
   if any (max 3: a brand color, a data-correctness rule, a format requirement). Adjectives set
   ambition; non-negotiables set correctness.
3. **Resources: inventory + scoped discovery** — name the verified, load-bearing tools/paths, sketch
   one example workflow, then release it ("you can accomplish this many ways"). Grant discovery in
   tiers: local resources first (the working directories in `environment.md`, existing project
   assets) → the owner's connected MCPs → curated web (official docs, established well-maintained
   libraries, properly licensed assets). Never a blanket "the internet is available to you" — the
   fresh session hunts, but in named waters.
4. **Creative freedom + decision authority** — explicit permission to deviate, swap named tools for
   better ones it finds, and settle judgment calls (naming, styling, scope edges, tradeoffs) with
   taste instead of deferring. Never skip this part.
5. **Verification loop** — a concrete acceptance spec, never a vibe: name the artifact that gets
   inspected, the specific checks it must survive, and what failure looks like. At least three passes
   adapted to the medium (render and watch, load and click, run on real input) — and at least one
   pass must be allowed to conclude "the concept is weak, restart" (a loop that can only make small
   fixes rubber-stamps mediocrity). Close with the **owner gate**: before declaring done, the session
   simulates the owner's review — given their standards, taste, and perceptive eye, would THIS
   survive them? Grade against "would the owner ship it," never against "competent." Not a confident
   yes → iterate (structural changes allowed, cap ~3 loops), then surface honestly what's still
   short. **In the walkthrough, 5/7 presents the actual planned test — artifact, checks, failure bar
   — so the owner confirms the test itself; this is where check-your-work ambiguity dies.**
6. **Delivery + review gate** — exactly where results land (environment defaults unless the ramble
   names a destination). If the output leaves the machine or reaches another person (deploy, publish,
   post, send), or overwrites something the owner cannot easily get back (editing originals in place,
   bulk renames across a real archive): the prompt requires preview links/files for the owner's
   review and forbids autonomous publishing or destructive writes — verbatim clause in
   `environment.md`.
7. **Goal line + autonomy + blocked-path + model routing** — restate the deliverable as one sentence:
   "[X with Y and Z] is your goal." Then the autonomy directive with the blocked-path clause from
   `environment.md` verbatim. Parallelization nudge only when the job is genuinely parallel (many
   independent items), sized to the task. Whenever the goal will spawn agents, the prompt MUST carry
   the burn-posture block from `environment.md` — main-loop routing, context discipline (exploration
   to subagents, phase-scoped context), right-sized model per job, finisher-scoped briefs over
   from-scratch fleets, diagnose-before-escalating.

## Red flags — wrong shape

- A finished prompt in your first reply → you skipped the walkthrough. The walkthrough IS the skill.
- An assumption in the final prompt that never appeared on an Inferred: line → back it out or surface
  it.
- Asking the owner questions before a draft exists → draft first; they react, they don't generate.
- A part presented without the menu under it → they lose `back`, `restart`, `pause` and `abandon`
  without knowing it.
- Demanding the literal keyword when they clearly meant a verb → infer it and move on.
- Reading a multi-character reply by its first letter → "change" is not `c`, and "back to what you
  said about resources" is not `b`. One character, or it's prose.
- `abandon` firing on the keystroke without a confirm → it is the only verb that destroys work.
- A `change` that lands on another part without announcing the move → silent navigation is the thing
  to avoid, not navigation itself. One line, then go.
- Refusing a request whose destination and edit you are both holding → that is a tool correcting the
  owner's grammar. Route it and say so.
- Routing on naming that two parts could answer to → an announced guess is still a guess. Print the
  candidate picker instead.
- A candidate picker that leaves out the part on screen → `back`'s backward-only rule leaking into a
  question that isn't about going back. List every part that answers to the phrasing.
- Inventing the edit when prose named a destination but carried no content → route, then ask.
- Returning to the frontier while a stale part sits unresolved behind it → stale first, always.
- Reading ambiguous prose ("cancel this") as `abandon` without asking → generous inference is right
  everywhere else and wrong on the one irreversible verb.
- Rippling a part that doesn't depend on what changed → a wording tweak costing six re-drafts is how
  the walkthrough gets long enough that the owner reaches for the fatigue valve.
- Re-presenting downstream parts on a first forward pass → they were never confirmed; there is
  nothing to protect and nothing to re-confirm.
- A resumed goal left marked `paused` on disk → the file is the source of truth, and it is lying.
- A restart that discards the old ramble → `## Superseded rambles` is how a resume understands what
  changed.
- Re-presenting parts the ripple pass didn't actually change → the point of the diff is that they
  confirm each thing once.
- Continuing a resumed goal without re-running the `## Verified` checks → you are naming Monday's
  paths on Thursday's machine.
- "Ask me if anything is unclear" inside the generated prompt → kills the autonomous run; the
  blocked-path clause already covers it.
- A hosted deploy (Netlify etc.) as a default destination → defaults come from `environment.md`;
  deploys only when the owner names one.
- A verification part with no named artifact, checks, or failure bar → that's a vibe, not a test.
- A generated prompt that spawns agents with no model routing → top-tier everywhere is a sledgehammer
  driving finishing nails.
- An autonomous goal delivered without the Launch line → the run defaults to whatever's configured,
  which may be the expensive option for work that doesn't need it.
- An Expect block with point estimates instead of bands → false precision on a number you cannot
  measure.
- The Expect block pasted inside the generated prompt → it is for the owner's launch decision, not
  for the fresh session.
- Fleet of from-scratch agents where gap-fill/finisher briefs would do → scope-per-agent is a bigger
  burn lever than model tier.
- "Done" graded against competent/correct instead of "would the owner ship it" → this is the single
  most common way these runs fail. Competent is the floor, not the bar.
