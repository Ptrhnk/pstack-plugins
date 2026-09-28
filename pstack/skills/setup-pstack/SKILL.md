---
name: setup-pstack
description: Configure which Claude model pstack uses per role. Offers presets, lets the user override any role, and writes a config file that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.claude/pstack-models.md`, the file every pstack skill reads to pick its model per role.

## Steps

### 1. Detect available models

The values the Agent tool's `model` parameter accepts in this session are the dependable source (typically `opus`, `sonnet`, `haiku`, and `fable`). Read them from the tool's schema. Never write a value you have not confirmed is accepted. The alias `inherit` is always valid. It means the role runs on the parent chat model, so the skill omits `model` on the Agent call.

### 2. Load current state

The default role-to-model mapping is the file shape shown in step 5 below. If `~/.claude/pstack-models.md` already exists, read it and treat its `# profile` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Profile, map, and confirm

**(a) Ask for a profile.** Use AskUserQuestion. Offer these four options with these exact labels, and name the current profile when the file records one.

- `default — opus for code, reviews, judgment; sonnet for wide fan-outs`
- `max — opus everywhere`
- `lean — sonnet for code and fan-outs; opus for reviews, judgment, hardest tasks`
- `custom — pick every role`

**(b) Apply it.** Build the working table from the step 5 defaults, which are the `default` profile. On a re-run keep any role the user set by hand. `max` sets every role to `opus`. `lean` sets `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`, `how explorer`, `why investigators`, and `swarm workers` to `sonnet`, and keeps the rest at `opus`. `custom` starts from the current table and goes straight to (c).

**(c) Show the roles and confirm.** Show every role with its model and a few words on what the role does, so the user can judge where a stronger model pays off. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit` as the options. Use AskUserQuestion. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, `inherit` entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it, preferring one that differs from the parent's model. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

The role groups, for the explanation in (c):

- **Code delegates.** `feature, refactoring`, `bug-fix`, `perf-issue`, `hillclimb`. The subagent that writes the diff.
- **Hardest tasks.** Cross-cutting design, gnarly concurrency, subtle algorithms.
- **Judgment and prose.** Plans, PR bodies, replies, synthesis.
- **Reviews and panels.** `interrogate reviewers`, `arena runners`, `arena cross-judge pool`, `architect runners`, `reflect ...`. Adversarial review and design bakeoffs.
- **Wide fan-outs.** `how explorer`, `why investigators`, `swarm workers`. Many read-mostly agents at once, where cost and rate limits add up fastest.

Reasoning effort is not a per-role setting. Subagents run at the session's effort level (`/effort` or the `effort` setting). Say so if the user asks for per-role effort.

### 4. Validate

Every value written must be a detected model or `inherit`. If a chosen value is not available, stop and ask again.

### 5. Write the file

Write `~/.claude/pstack-models.md` with a `# profile` line with the chosen label and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit` as a value: the role runs on the parent chat model (omit the Agent `model`). `inherit` entries in a panel list still count toward its fan-out.
# profile: default
feature, refactoring: opus
bug-fix: opus
perf-issue: opus
hillclimb: opus
judgment and prose: opus
hardest tasks: opus
how explorer: sonnet
how explainer: opus
why investigators: sonnet
why synthesizer: opus
reflect tooling: opus
reflect judgment, divergent, synthesizer: opus
arena runners: opus, opus, opus
arena cross-judge pool: opus
swarm workers: sonnet
architect runners: opus, opus, opus
interrogate reviewers: opus, opus, opus
```

### 6. Confirm

Tell the user the file was written and that skills read it on their next spawn. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill`. On no, move on without pushing.
