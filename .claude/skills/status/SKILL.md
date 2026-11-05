---
name: status
description: Brief the user on every Herdr workspace ranked by what needs their attention and regenerate TODO.md. Use when the user asks for status, what is next, what needs attention, or on a /loop tick. Also use when the user wants to park, snooze, annotate or silence a workspace or PR.
---

## Briefing

1. Run `bin/herdr-status --json`. This refreshes `state/items.json` and applies `state/overrides.toml`.
   Report any `warnings` (unreachable machines, override typos) first.
2. For every NOW and SOON item, check the reason before reporting it:
   - CI failing: `gh pr checks <n> --repo <owner/repo>`, then `gh run view <id> --log-failed` for the
     first failing job; name the job and the first real error line.
   - New feedback: `gh pr view <n> --repo <owner/repo> --comments`; say who asked for what.
   - Finished or blocked session: use `summary`, `last_prompt` and `last_reply` from the JSON; read
     the screen with `herdr agent read` only if those do not explain it.
3. Run `bin/herdr-status --todo` to regenerate `TODO.md`. Never edit `TODO.md` by hand: everything
   except its Notes section is rewritten on every render, and Notes belong to the user. Read Notes
   when ranking (e.g. "parked") and suggest an override if a note should change the ranking.
4. Reply with the briefing: the top items with one concrete next action each, suggested prompts for
   sessions that need one, and a one-line count of everything else, including how many are parked.
   Do not repeat the whole board.

## Overrides

When the user says to park, snooze, annotate or stop flagging something, edit `state/overrides.toml`
(the header comment documents the format), then run `bin/herdr-status --todo` and check its warnings.

- Key by the workspace label exactly as the board shows it (remote: `host:label`), or by PR number
  (`owner/repo#n` if the number is ambiguous). Prefer the PR key when the user names a PR.
- `parked = true`, `snooze_until = YYYY-MM-DD` (until that day starts), `ignore_question = true`,
  `note = "..."`. Convert relative dates ("until Monday") to an absolute date and say which one.
- Remove an entry when the user unparks something; drop expired snoozes when you see them.
- "X is related to / handled in workspace Y": add the issue to `links` of `[workspace."Y"]`.
- "I already did X" for an inbox item: `done = <today>` on its key, so new activity resurfaces it.

Stay read-only towards the workspaces (see CLAUDE.md): suggest prompts for other sessions, never send them.
