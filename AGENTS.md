# AGENTS.md

## Agent skills

Skills come from [mattpocock/skills](https://github.com/mattpocock/skills), installed into `.claude/skills/` with `npx skills add mattpocock/skills` and pinned in `skills-lock.json`. Do not edit them locally; update with `npx skills update`.

### Issue tracker

Issues and specs live as GitHub issues in this repo, via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Domain docs

Single-context: `CONTEXT.md` at the repo root, ADRs in `docs/adr/`. Both are created lazily by `domain-modeling`. See `docs/agents/domain.md`.

While working on the template itself (`.template/` exists), the template's own glossary and ADRs live in `.template/CONTEXT.md` and `.template/docs/adr/` instead, so they are not copied into new projects (`.template/` is removed in a copy on its first session). Read and enforce them exactly as you would the root ones.

## Navigation (repo-local)

This repo may be driven by a non-engineer. Never assume the user knows which slash command to run.

- The `next-step` skill (`.claude/skills/next-step/`, repo-local, not from mattpocock/skills) diagnoses where the work currently stands and proposes exactly one command to run next.
- A `SessionStart` hook in `.claude/settings.json` loads `docs/agents/flow-map.md` into context at the start of every session.
- Before writing implementation code for a request that has not been through the flow, run `next-step` first. Skip it only if the user explicitly says to go straight to code.
- Session ceremonies: `s` reports where the last session left off (run it at the top of the first reply of every session, unprompted); `f` drives the repo to a state where the container can be destroyed safely and then declares, with evidence, whether it is safe to close. Both are repo-local skills, triggered by the bare letter or the slash command.
- Never schedule timed check-ins on yourself (`send_later`, `create_trigger`, `/loop`, `ScheduleWakeup` or similar), including the default "safety-net check-in" after opening a PR. Watch PRs only through event delivery (`subscribe_pr_activity`). The only exception is a check-in the user explicitly asks for in the moment.
- `docs/agents/handover.md` carries state across sessions: the container is ephemeral, so anything not written there is lost. Append a new entry at the top when work is shipped or a direction is agreed, run `.claude/hooks/handover-trim.sh`, and commit it immediately. `.claude/hooks/session-start.sh` owns the retention and read-window numbers; don't restate them elsewhere.

## Development flow

### Starting new work, or adding a feature to an existing codebase

Talk it through in the conversation -> `/conduct`

There is no interview skill. The user describes what they want in plain words; once it is settled, `/conduct` takes over.

`/conduct` (repo-local, `.claude/skills/conduct/`) runs everything after that conversation without asking the user anything except at the mid-point check and the final check: recording vocabulary and decisions, spec, design review, tickets, implementation, final review, fixes, and merge on GO. Claude does every stage itself in this session; reviews run in fresh sub-agents so they do not inherit the author's assumptions.

The manual path still works: conversation -> `/to-spec` -> `/to-tickets` -> `/implement`

Vocabulary and decisions settled in the conversation are recorded with `/domain-modeling` into `CONTEXT.md` and `docs/adr/` (`/conduct` does this as its first step), so they do not evaporate with the session. Those files are created lazily, so there is nothing to write and nothing to skip when a repo is still empty.

`/implement` drives `/tdd` at pre-agreed seams: red -> green, one behaviour at a time, tested through the public interface rather than implementation details, one vertical slice per loop. It closes with `/code-review` before committing.
