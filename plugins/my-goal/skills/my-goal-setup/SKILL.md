---
name: my-goal-setup
description: Use when the user invokes /my-goal-setup, asks to set up or reconfigure the my-goal skill, or when /my-goal runs and ~/.claude/my-goal/environment.md does not exist. Detects the user's environment, asks only what it cannot detect, and writes their personal my-goal configuration.
---

# my-goal setup

Write `~/.claude/my-goal/environment.md` — the ground-truth file that `/my-goal` reads to produce
goal prompts naming this user's real paths, real tools, and real taste.

**Detect first, ask second.** You can read their MCP config, probe for binaries, and find their notes
vault. Anything you can look up, look up — then confirm it. Asking a question you could have answered
yourself is the failure mode of this skill.

**Config lives outside the plugin on purpose.** Plugins install to version-numbered directories, so a
config written inside the plugin dies on the next update. `~/.claude/my-goal/environment.md` survives.
Never write config into the plugin directory.

## Step 1 — Detect

**Establish the platform first.** Every path below depends on it, and two of the three platforms are
not macOS. Your harness reports the OS in its environment context — use that. Otherwise infer it from
which home directory exists: `/Users/<name>` macOS · `/home/<name>` Linux · `C:\Users\<name>` Windows.

| | macOS | Linux | Windows |
|---|---|---|---|
| Claude config | `~/.claude/` | `~/.claude/` | `%USERPROFILE%\.claude\` |
| Obsidian config | `~/Library/Application Support/obsidian/obsidian.json` | `~/.config/obsidian/obsidian.json` | `%APPDATA%\obsidian\obsidian.json` |
| Big / external storage | `/Volumes/*` | `/media/*`, `/mnt/*` | drive letters past `C:` |

**Read config files with your own file tools. Do not shell out to parse them.** `jq` ships with
neither macOS nor Windows, so a `jq` pipeline returns nothing on a stock machine, writes a thin
config, and reports no error — the worst possible outcome, because it looks like it worked. Read the
JSON and pull the keys yourself:

- `~/.claude.json` → `mcpServers` keys — the MCP servers they have configured
- the Obsidian config **for their platform** (table above) → `vaults[].path`
- `~/.claude/plugins/installed_plugins.json` → `plugins` keys — what else they run
- `~/.claude/skills/` → directory listing — their personal skills
- `~/.claude/CLAUDE.md` → read it; it often states how they want to be addressed and how they work

Also check the live session's own tool list for connected MCPs — more current than the config file.

**Candidate working directories** — list what actually exists, don't assume: `~/Projects`,
`~/Developer`, `~/Code`, `~/src`, `~/Documents`, plus the platform's big-storage location.

**Binaries are the one probe that genuinely needs a shell.** Use the form matching the platform:

```bash
# macOS / Linux / WSL / Git Bash
for b in ffmpeg yt-dlp gs pandoc magick convert rg jq sqlite3 python3 node; do
  command -v "$b" >/dev/null 2>&1 && echo "$b"
done
```

```powershell
# Windows PowerShell — Claude Code's shell when Git for Windows is absent
foreach ($b in 'ffmpeg','yt-dlp','gs','pandoc','magick','rg','jq','sqlite3','python','node') {
  if (Get-Command $b -ErrorAction SilentlyContinue) { $b }
}
```

**Say the platform and any gap out loud before you ask anything.** One line is enough: *"Detected
Windows; found 3 MCP servers and one Obsidian vault; couldn't probe local binaries — no bash
available."* Missing results are fine. **Silently** missing results are not: a thin config the owner
knows about is fixable, a thin config that looked successful is not.

**If `~/.claude/my-goal/environment.md` already exists**, read it first. This is a reconfigure, not a
fresh install: carry forward everything they already set, present current values as the defaults, and
only rewrite what they change.

## Step 2 — Ask what you couldn't detect

Use one `AskUserQuestion` call with these four. Offer detected values as the first option so the
common case is one click, not typing.

1. **Name for the owner gate.** Pre-fill from `git config user.name` or `CLAUDE.md`. Explain in the
   question what it's for: before declaring a goal done, the fresh session simulates *their* review
   and asks "would this survive them?" — so the name should be what they'd call themselves.
2. **Plan tier — Max / Pro / API or other.** This changes the generated output more than any other
   answer; see Step 3. It also sets the units `/my-goal` uses when it estimates what a goal will
   cost to run — a fraction of a usage window for Max and Pro, a dollar band for per-token billing.
3. **Where heavy deliverables land.** Offer detected candidates (external volumes, `~/Projects`,
   `~/Developer`). This is where builds, renders, and working files go.
4. **Where notes and docs land.** Offer detected Obsidian vaults first if any exist, then
   `~/Documents`. If they have no notes system, "same as deliverables" is a fine answer.

Don't ask about the review gate — default it ON (nothing gets published, posted, deployed, or sent
without their preview) and tell them in the summary how to turn it off. It's the safe default and a
question nobody benefits from being asked.

Don't ask about brand or style touchpoints. Leave that section as a commented stub they can fill in
later; most people won't need it and it's the wrong thing to interrogate a new user about.

## Step 3 — Tier branching

Read `environment.template.md` (next to this skill) and fill it in. The model-routing section is the
one part that genuinely differs by tier — write the branch that matches their answer and delete the
other. Do not ship both and tell them to pick.

**Max** — multiple model tiers available, so route by job:
- Top tier for concept generation, taste calls, and the final owner-gate review — dispatched
  sparingly from a cheaper main loop.
- Mid tier for production builds from a clear spec, finishing work, structured verification.
- Small tier for mechanical checks, file ops, renders, format validation — anything with an
  objectively checkable output.
- The launch line names the cheaper capable model for the main loop, reserving the expensive one for
  subagent moments where it earns the premium.

**Pro** — one working model in practice, so the ladder is noise. Cut it. What matters more here, not
less, is context discipline, because usage limits bite sooner:
- Push exploration, research, and bulk file reading into subagents; their context dies with them and
  only the conclusion comes back.
- Keep large file contents out of the main loop.
- For multi-phase goals, write state to disk at phase boundaries so a fresh lean session can pick up,
  rather than one long session dragging its whole history through every turn.
- The launch line is plain `claude` with no `--model` flag.

**API or other** — same structure as Max, but the launch line stays generic since their entry point
may not be the `claude` CLI at all. Note that cost is per-token and direct, so the routing advice
applies literally.

**Session retention ships pre-filled.** The template carries `Keep abandoned goals for: 30 days`
with a comment explaining it. Don't ask — same reasoning as Watch level: a new install has no
abandoned goals to have an opinion about, and the comment tells them how to change it. Just name it
in the Step 4 summary so they know it exists.

**Fill the cost posture in the same pass.** `{{PLAN}}` is their answer; `{{BINDING_WINDOW}}` follows
from it — `weekly (the 5-hour window rarely binds at this tier)` for Max, `5-hour (it binds well
before the weekly one)` for Pro, `none — billed per token` for API or other. Leave **Watch level** at
`relaxed`. Don't ask about it; a new install has no history of runs dying on a limit, and the comment
in the template explains how to change it once they do.

Universal, ships in all three: right-size the model to the job; prefer narrow finisher and gap-fill
briefs ("these files exist, complete X, one closing pass") over from-scratch fleets, because
scope-per-agent moves cost more than model tier does; and diagnose before escalating — an ambiguous
spec, too little effort, and a genuine capability ceiling all look like "bad output" and only the
third one is fixed by a bigger model.

## Step 4 — Write, summarize, verify

1. Create `~/.claude/my-goal/` and write `environment.md` **with your file tools** — `mkdir -p` is
   POSIX-only and fails in PowerShell. On Windows that path is `%USERPROFILE%\.claude\my-goal\`.
2. Summarize in plain language what you recorded — the name, the two directories, the tier, the tools
   you found, and that abandoned goals are kept 30 days by default. Six lines, not a wall.
3. Tell them the file is theirs to edit freely, that `/my-goal` never needs changing when it does,
   and that plugin updates won't touch it.
4. Offer a smoke test: "give me a one-line description of something you want built and I'll run
   `/my-goal` on it." An install that has never produced a prompt isn't verified.

## Red flags

- Parsing config by shelling out to `jq` → it is on neither a stock macOS nor Windows. Read the file
  with your own tools. A `jq` miss is silent, which makes it worse than a crash.
- Hardcoding `~/Library/Application Support` or `/Volumes` → macOS-only. Branch per the Step 1 table.
- Reporting a successful setup without naming the platform → the owner cannot tell a real detection
  from a total miss, and neither can the next session.
- Asking for a path you could have detected → detect, then confirm.
- Writing config into the plugin directory → it dies on update. `~/.claude/my-goal/` only.
- Shipping both tier branches with "pick one" → you asked the tier question; act on the answer.
- A tool roster listing things you never verified exist → `/my-goal` will name them in a prompt and
  the fresh session will fail reaching for them. Only record what the detection actually found.
- Interrogating a new user through a long questionnaire → four questions, detected defaults, done.
- Leaving `{{PLAN}}` or `{{BINDING_WINDOW}}` unfilled → `/my-goal` falls back to bare
  light/moderate/heavy with no units, and nags them to re-run setup.
- Asking them to set Watch level → they have no history yet. Default it and let the template explain.
- Asking them how long to keep abandoned goals → same answer. It ships at 30 days with a comment.
- Dropping the retention line from the written config → `/my-goal` then falls back to 30 days
  silently, which is right, but the owner never learns the setting exists.
