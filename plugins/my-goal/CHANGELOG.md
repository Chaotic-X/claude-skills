# Changelog — my-goal

## 1.2.1

**Candidate picker.** Disambiguation reused the `back` picker, which by definition lists only the
parts *behind* the cursor. So when an ambiguous naming could mean the part on screen — frequently the
strongest reading — the picker meant to resolve the ambiguity structurally could not offer the likely
answer, and steered toward an earlier part instead. A picker whose options exclude the right one
looks like a question and behaves like an answer.

- **The candidate picker is now distinct from the `back` picker.** `back` moves backward, so its
  picker stays backward-only. Disambiguation asks *which part did you mean*, so it lists every part
  that answers to the phrasing, wherever it sits, and marks the current one:
  `did you mean: (2) quality bar · (5) verification loop — on screen now`. Parts *ahead* of the
  cursor stay out either way — they are unpresented drafts, so the owner cannot have meant one.
- **Resolution now precedes the on-screen rule.** The two were unordered, and both fired on the same
  input: "the named part is the one on screen, so change it here" and "two parts could answer to
  this, so print the picker." Resolution comes first — deciding a phrase must mean the part in front
  of you because that is where you are standing is the same announced guess by a quieter route.
- **Red flag added** for a candidate picker that omits the part on screen: `back`'s backward-only
  rule leaking into a question that is not about going back.

Found by behavioural runs against 1.2.0 — six scripted walkthroughs, one per routing rule. The
routing rules themselves passed; the picker they delegate to did not.

## 1.2.0

**Walkthrough controls.** The confirm menu gained single-letter shortcuts, `restart`, and
`abandon`, and lost `adjust`.

- **Single-letter shortcuts.** `y` `c` `b` `r` `p` `a`. A key *opens* the action rather than firing
  it — only `y` completes on its own; every other key asks the question its verb implies and waits.
  A lone character is a shortcut; anything longer is read as prose, so "change the resources part"
  never collides with `c`.
- **`adjust` and `correct` merged into one verb, `change`.** The two differed only in whether the
  ripple pass ran, which made the owner classify their own edits — and the classification was
  already model-inferred whenever they spoke prose instead of typing a keyword. Now one verb, and
  the ripple's diff decides what actually went stale. Renamed to `change` rather than kept as
  `correct` because the merged verb covers preference edits too ("make it warmer" is not an error
  being flagged), and a label implying fault would make exactly those edits hesitate. Merging also
  frees `a` for abandon.
- **`restart`** rebuilds the walkthrough from a new or revised opening statement, in the same
  session file. The old ramble moves to a `## Superseded rambles` section rather than being lost.
- **`abandon`** ends a goal for good, mid-walkthrough or from the Phase 0 list. It is the only
  destructive verb, so it confirms first and names what will be lost.
- **Retention sweep.** Abandoned goals are kept for a window set in `environment.md`
  (`Keep abandoned goals for`, default 30 days when absent), then deleted during the next Phase 0
  scan — leaving one line in `sessions/_abandoned-ledger.md`. Delivered goals never age out.
- **No silence on unmapped keys.** A stray character comes back with a best reading and a question
  instead of nothing.

**Step scoping.** A bare `c` is scoped to the part on screen — the key names nothing, so the current
part is the only referent. Prose that explicitly names an earlier part is routed as a `back` carrying
the change with it, announced in one line before the move rather than refused or performed silently.
And `back` now always establishes its destination: pressing `b` prints the parts behind the cursor on
one line, so it can never rewind somewhere the owner didn't name.

**Ripple scoping.** Merging the two verbs meant every edit triggered a downstream
re-draft, so the ripple pass now scopes itself: a part ripples only if it actually depends on what
changed, a first forward pass re-drafts silently instead of re-presenting parts that were never
confirmed, and an explicit "just this, don't touch anything else" suppresses the ripple outright.

**Review gate broadened.** The standing clause in `environment.template.md` now triggers on output
that leaves the machine *or reaches another person* — not only output that goes public — and says
explicitly that writing a file for the owner, or sending something only to them, is not publishing.
Part 6 of the anatomy matches, and also covers writes the owner cannot easily undo (editing
originals in place, bulk renames across a real archive).

**`restart` confirms too.** It discards every confirmed part, so when any are confirmed it names the
loss before clearing — the same courtesy `abandon` gets. With nothing confirmed, it just goes.

**Backward compatible.** Session files written under 1.1.0 open and resume unchanged; a
`confirmed: adjusted` or `confirmed: corrected` marker is read as `confirmed`. Full words and plain prose work exactly as
before — the letters are additive, never required syntax. A config with no retention line falls back
to 30 days silently.

## 1.1.0

Walkthrough controls, pause/resume, and the Expect block.
