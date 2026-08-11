---
name: my-goal
description: Use when the user invokes /my-goal, says "turn this into a goal prompt" / "write me a goal prompt", rambles a desired outcome and asks for the prompt that will make it happen in a fresh session, or asks to resume or pick up a goal prompt they paused earlier. The deliverable is one paste-ready goal prompt, never the build itself — do NOT use when they want the thing built right now in this session.
---

# Goal Prompt Writer (/my-goal)

Turn the user's ramble into one exceptional goal prompt for a fresh autonomous session — through a
draft-first walkthrough where every inference is surfaced for their confirm/correct. You are not
building the thing, and you are not interviewing them from scratch: you draft, they react.

Throughout this skill, **the owner** means the person named in `~/.claude/my-goal/environment.md`.
Use their actual name in what you write; "the owner" is a placeholder for you, not for them to read.

## Configuration

Ground truth lives at `~/.claude/my-goal/environment.md` — delivery defaults, standing clauses, tool
roster, model routing, cost posture.

**If that file does not exist, invoke the setup skill first**, let it finish, then continue from
Phase 1 with the ramble the user already gave you. The skill is named `my-goal-setup` when installed
into a skills directory, and `my-goal:my-goal-setup` when installed as a plugin — use whichever
appears in your available-skills list. Do not ask them to repeat themselves, and do
not try to run without a config — a goal prompt that names no real paths or tools is worthless.

In-progress and paused walkthroughs live in `~/.claude/my-goal/sessions/`. Both paths sit beside
`environment.md`, deliberately outside any version-numbered plugin directory, so updating the plugin
never destroys the owner's config or their unfinished goals.

## Process

### Phase 0 — Unfinished goals
Before anything else, check `~/.claude/my-goal/sessions/` for files whose `status:` is not
`delivered`. Distinguish two kinds:

- **paused** — the owner said `pause`.
- **interrupted** — `in-progress` from a session that ended without delivering (context clear, crash).

If any exist, offer them before starting fresh — one line each: slug, when, `position N/7`, and the
one-line desire from part 1. The owner picks one or says they want a new goal. If none exist, say
nothing about it and go straight to Phase 1.

**Resuming is warm.** Restore the confirmed parts and position, then re-read `environment.md` (it may
have changed) and re-run every check in the file's `## Verified` block. Anything that has vanished,
moved, or been renamed is presented as a **correction** on the part that named it — which marks that
part `stale` and ripples normally. Only then continue at `position`. A path verified last Monday is
not a path verified today.

### Phase 1 — Intake
Accept the ramble as-is; voice-to-text tolerant (interpret intent over literal words: "Quad MD"
means CLAUDE.md, "Netlefi" means Netlify). Read `~/.claude/my-goal/environment.md` once. Ask nothing
yet. Open the session file with the ramble stored verbatim — slug it from the deliverable in two or
three hyphenated words (`comfyui-style-sheets`, `rhune-landing-page`), so a list of paused goals is
readable without opening any of them.

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
yes · adjust <what> · correct <what>
back [n] · pause
```

Every fact in the drafted prose that came from neither the ramble nor a verified check gets an
Inferred line — no silent specifics. **Print the menu after every part**, identically, so the owner
never has to remember what is available.

**The five verbs.** What separates `adjust` from `correct` is the *source* of the change, not its
size — and it changes what you do next:

| Verb | Means | You do |
|---|---|---|
| `yes` | accept as drafted | mark `confirmed`, advance |
| `adjust <what>` | your draft was a fair read, they want it different | patch in place, mark `confirmed: adjusted`, **no ripple**, advance |
| `correct <what>` | one of your Inferred lines is **wrong** | fix the premise, mark `confirmed: corrected`, **run the ripple pass**, note if `environment.md` or your reading of the ramble is what was stale, advance |
| `back [n]` | return to part n | set n to `awaiting`, rewind position; run the ripple pass after they change it |
| `pause` | stop here | write `status: paused`, tell them the resume command, end the turn — no summary, no further questions |

**The keywords are optional.** Infer the verb from what they actually say. "No, it's ToT not SoO" is
a correct; "make it warmer" is an adjust; "go back to the resources part" is a back. The menu names
categories, not a syntax they must obey — this skill is voice-to-text tolerant everywhere else and
the menu is no exception.

**Ripple pass** (after any correct, or after a back-edit): re-draft every part downstream of the one
that changed, diff each against the text already confirmed, and mark only the ones that *materially
moved* as `stale`. Re-present the stale parts in order, labeled with what caused the change.
Parts that did not move keep their confirmation and are never shown again. Never silently revise a
part the owner already confirmed.

**Fatigue valve:** "rest looks good", "skip to the end", or anything similar batch-confirms
everything remaining — jump straight to Phase 4 with zero further questions.

**After every confirmed part, write the session file** (schema below) before presenting the next one.
The file is the source of truth for `back`, `pause`, `resume`, and the ripple diff — not your memory
of the conversation.

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
are the owner's record of what they have asked for before.

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
status: in-progress          # in-progress | paused | delivered
position: 5                  # part currently awaiting confirm
---

## Ramble
<the original ramble, verbatim — never rewritten, even after corrections>

## Verified
- /Volumes/x8/Claude_Resource_Dir/Alex/ — exists (16:55)
- ComfyUI MCP xSSD4:8188 — reachable (16:55)

## Parts

### 2 — Quality bar  [confirmed: corrected]
<drafted prose>
Inferred:
- style-locked across the set — CONFIRMED
Correction: "ToT not SoO" -> premise fixed, rippled to 5

### 5 — Verification loop  [awaiting]
<drafted prose>
```

Per-part status is one of `awaiting`, `confirmed`, `confirmed: adjusted`, `confirmed: corrected`,
`stale`. Keep the ramble verbatim — it is what a warm resume re-reads when the confirmed parts are
days old.

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
   names a destination). If the output is public-facing (deploy, publish, post, send): the prompt
   requires preview links/files for the owner's review and forbids autonomous publishing — verbatim
   clause in `environment.md`.
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
- A part presented without the menu under it → they lose `back` and `pause` without knowing it.
- Demanding the literal keyword when they clearly meant a verb → infer it and move on.
- Treating a correction like an adjustment → a wrong premise that never ripples poisons every part
  drafted on top of it.
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
