# claude-skills

Claude Code skills worth sharing.

## `/my-goal` — the goal prompt writer

You know what you want. Describing it well enough that Claude can go build it *unsupervised* is a
different skill, and it's the one that decides whether you get the thing or a plausible-looking near
miss.

`/my-goal` writes that prompt for you. You ramble; it drafts a complete goal prompt across seven
parts, then walks you through them one at a time — showing you every assumption it made so you can
change it before the assumption gets baked in. What comes out is one paste-ready prompt, on your
clipboard, for a fresh session to execute autonomously.

It does not build the thing. It writes the prompt that makes building it go well.

### What's in the seven parts

The anatomy is the point. A goal prompt that skips any of these tends to fail in a predictable way:

| Part | What it prevents |
|------|------------------|
| Desire + stakes | Vague deliverables, wrong quantity |
| Quality bar | "Technically correct, obviously mediocre" |
| Resources + scoped discovery | Aimless wandering, or refusing to look anything up |
| Creative freedom | Constant deferral instead of decisions |
| Verification loop | "Done" meaning "I stopped" |
| Delivery + review gate | Things getting published that you never saw |
| Goal line + autonomy | Drift, and burning your usage on the wrong model |

The verification part is the one that matters most, and it ends with a gate: before declaring
anything finished, the session simulates *your* review and asks whether the work would actually
survive you. Not whether it's competent — competent is the floor.

## Install

In Claude Code:

```
/plugin marketplace add https://github.com/Chaotic-X/claude-skills.git
/plugin install my-goal
```

> Use the full URL as written. The shorthand form `Chaotic-X/claude-skills` also works, but it
> clones over SSH and will fail if you haven't set up GitHub SSH keys. The URL above uses HTTPS and
> works for everyone.

Then configure it for your machine:

```
/my-goal-setup
```

Setup looks at your actual machine — your MCP servers, your installed tools, your notes vault if you
have one — and asks you about four things it can't figure out on its own. Takes about a minute.

Then just use it:

```
/my-goal
```

…followed by whatever you want, however you want to say it. Rambling is fine. Voice-to-text mangling
is fine; it's built to interpret intent over literal words.

If you skip setup and run `/my-goal` first, it'll run setup for you and then carry on. Nothing is
lost.

### While it's walking you through

After every part you get the same menu. `y` and `p` act immediately; the other four open the action
and ask you something first, so nothing destructive ever happens on a single keystroke.

```
(y)es · (c)hange <what> · (b)ack [n]
(r)estart · (p)ause · (a)bandon
```

`change` covers anything you want different about the part in front of you, from a wrong assumption
to "make it warmer". `back` returns to an earlier part — press it bare and it shows you which ones
you can reach. `restart` rebuilds the whole thing from a new opening statement, for when the premise
was wrong rather than one part. `pause` stops and keeps your place; resume whenever. `abandon` ends a
goal for good — it confirms first, then keeps the file for 30 days by default before deleting it,
leaving one line as the record.

You never have to use the keywords. Plain speech lands where you'd expect: "make it warmer", "go back
to the resources part", "forget this whole thing".

## Your configuration

Setup writes one file:

```
~/.claude/my-goal/environment.md
```

That's yours. Edit it whenever — change where deliverables land, add tools you've installed since,
add brand rules for a recurring project. The skill reads it every run and never needs modifying
itself.

It deliberately lives **outside** the plugin directory, because plugin updates install to a new
folder each time. Anything kept inside a plugin gets left behind on update; your config won't be.

Re-run `/my-goal-setup` any time to reconfigure — it reads your existing settings and only changes
what you tell it to.

## Updating

```
/plugin update my-goal
```

Your `environment.md` is untouched.

## A note on plan tiers

Setup asks whether you're on Max, Pro, or API, and it genuinely changes what gets written.

On **Max**, generated prompts route work across model tiers — the expensive model for concept and
taste and the final review, cheaper ones for spec-driven building and mechanical checks.

On **Pro**, that ladder is mostly fiction, so it's cut. What replaces it is context discipline, which
matters *more* on Pro rather than less, because limits arrive sooner: push exploration and bulk
reading into subagents so their context dies with them, keep big files out of the main conversation,
and checkpoint long jobs to disk instead of running one enormous session that re-reads its entire
history every turn.

## Requirements

Claude Code. macOS or Linux. `jq` makes setup's detection step better but isn't required.

## License

MIT
