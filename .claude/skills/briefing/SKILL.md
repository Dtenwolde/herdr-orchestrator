---
name: briefing
description: Fill in the Focus today section of today's briefing file. The board watcher asks for this on weekdays at 09:00 or when the user opens the orchestrator; also use it when the user asks for a briefing or a plan for the day.
---

1. Find today's newest briefing: `ls -t briefings/$(date +%F)*.md | head -1`. If there is none, run
   `bin/herdr-status --briefing --cached` to write it.
2. Read it. Its Since and Open sections come from the board; do not change them.
3. Run `bin/herdr-status --json --cached` and check the NOW, SOON and INBOX items the way the status
   skill does (failing job and first error line, who asked for what, what a finished session needs).
   Read the user's Notes in `TODO.md` and the notes in `state/overrides.toml` for context.
4. Replace the placeholder under `## Focus today` with 3 to 7 bullets, most important first:
   `- [workspace or repo#n] the concrete next action (why, since when)`. Mention snoozes that end
   today or tomorrow. Touch nothing else in the file.
5. Reply in the chat with the same focus list and a one-line count of the rest.

Stay read-only towards the workspaces (see CLAUDE.md).
