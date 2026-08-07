---
name: my-goal
description: Use when the user invokes /my-goal, says "turn this into a goal prompt" / "write me a goal prompt", or rambles a desired outcome and asks for the prompt that will make it happen in a fresh session. The deliverable is one paste-ready goal prompt, never the build itself — do NOT use when they want the thing built right now in this session.
---

# Goal Prompt Writer (/my-goal)

Turn the user's ramble into one exceptional goal prompt for a fresh autonomous session — through a
draft-first walkthrough where every inference is surfaced for their confirm/correct. You are not
building the thing, and you are not interviewing them from scratch: you draft, they react.

Throughout this skill, **the owner** means the person named in `~/.claude/my-goal/environment.md`.
Use their actual name in what you write; "the owner" is a placeholder for you, not for them to read.

## Configuration

Ground truth lives at `~/.claude/my-goal/environment.md` — delivery defaults, standing clauses, tool
roster, model routing.

**If that file does not exist, invoke the setup skill first**, let it finish, then continue from
Phase 1 with the ramble the user already gave you. The skill is named `my-goal-setup` when installed
into a skills directory, and `my-goal:my-goal-setup` when installed as a plugin — use whichever
appears in your available-skills list. Do not ask them to repeat themselves, and do
not try to run without a config — a goal prompt that names no real paths or tools is worthless.

## Process

### Phase 1 — Intake
Accept the ramble as-is; voice-to-text tolerant (interpret intent over literal words: "Quad MD"
means CLAUDE.md, "Netlefi" means Netlify). Read `~/.claude/my-goal/environment.md` once. Ask nothing
yet.

### Phase 2 — Draft all 7 parts
Draft every part of the anatomy below immediately, filling gaps with the most probable reading of the
ramble plus environment defaults. Verify before you name: anything you plan to name in the prompt
(path, skill, MCP) gets a 30-second existence check (`ls` the path, check the tool list) — the live
environment is the source of truth, not the roster file. Name only what you verified AND what is
load-bearing (usually 2–4 things).

### Phase 3 — Walkthrough (the point of this skill)
Present the draft one part at a time, in order:

> **N/7 — [Part name]**
> "…drafted prose for this part…"
> Inferred: [one line per assumption — and what changes if it's wrong]

Every fact in the drafted prose that came from neither the ramble nor a verified check gets an
Inferred line — no silent specifics.

Wait for confirm/correct after each part. Fold corrections in on the spot; if a correction ripples
into later parts (scale, destination), revise them and say so. **Fatigue valve:** "rest looks good",
"skip to the end", or anything similar batch-confirms everything remaining — jump straight to Phase 4
with zero further questions.

### Phase 4 — Assemble + deliver
Weave the confirmed parts into flowing first-person prose (the owner speaking to the fresh session),
150–350 words, no headers or bullets inside the prompt. Deliver exactly this: the prompt in one
fenced code block, copy it to the clipboard (`pbcopy` on macOS, `xclip -selection clipboard` or
`wl-copy` on Linux — tell them it's on the clipboard), then one **Launch** line — the recommended
command for the fresh session, taken from the launch guidance in `environment.md` — then an
**Assumptions** list covering ONLY parts batch-confirmed via the fatigue valve — walked-through parts
are confirmed facts, not assumptions. Nothing else.

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
- "Ask me if anything is unclear" inside the generated prompt → kills the autonomous run; the
  blocked-path clause already covers it.
- A hosted deploy (Netlify etc.) as a default destination → defaults come from `environment.md`;
  deploys only when the owner names one.
- A verification part with no named artifact, checks, or failure bar → that's a vibe, not a test.
- A generated prompt that spawns agents with no model routing → top-tier everywhere is a sledgehammer
  driving finishing nails.
- An autonomous goal delivered without the Launch line → the run defaults to whatever's configured,
  which may be the expensive option for work that doesn't need it.
- Fleet of from-scratch agents where gap-fill/finisher briefs would do → scope-per-agent is a bigger
  burn lever than model tier.
- "Done" graded against competent/correct instead of "would the owner ship it" → this is the single
  most common way these runs fail. Competent is the floor, not the bar.
