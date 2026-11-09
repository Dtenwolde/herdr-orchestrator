# orchestrator

A live priority board for everything running in [Herdr](https://herdr.dev): every workspace, every
coding agent, their git state, their GitHub PRs and your GitHub inbox, ranked by what needs you next.
A read-only Claude Code session sits next to it to dig into the details, keep a `TODO.md`, and write
you a briefing every weekday morning.

```
 ● 1 now  ● 2 soon  ◆ 2 inbox  ● 2 running  ● 3 waiting  ● 1 cleanup
 ○ 1 parked · 9 quiet

── NOW ─────────────────────────────────────────────────── 1 ──
▌fix-null-join                                              12m
▌  #412 ✗ fix/null-join
▌  ↗ https://github.com/acme/widgets/pull/412
▌  ✗ CI failing (2: linux-debug, sanitizer)

── SOON ────────────────────────────────────────────────── 2 ──
▌parser-cleanup                                              3h
▌  #398 ✓ parser-cleanup
▌  ↗ https://github.com/acme/widgets/pull/398
▌  ✎ new feedback from alice (3h ago)

── INBOX ───────────────────────────────────────────────── 2 ──
▌↻ acme/widgets#405                                          1d
▌  Faster hash aggregate for wide keys
▌  ↗ https://github.com/acme/widgets/pull/405
▌  review requested by bob

── RUNNING ─────────────────────────────────────────────── 2 ──
▌docs-refresh                                               now
▌  ◐ agent working: Rewrite the install guide
▌    ↳ buildbox docs-refresh                                 2m
```

## What you get

- **The board** (`bin/orch-watch`): a live pane that refreshes on Herdr events within about two seconds,
  ranks every workspace as NOW / SOON / RUNNING / WAITING / CLEANUP / PARKED / QUIET, and shows why.
  It covers agent state (blocked, finished, working), git (dirty, unpushed), PRs (CI with progress,
  new review feedback, changes requested, approved, merged), and review worktrees for other people's
  PRs (re-review after new commits). PR and issue links are clickable.
- **Your GitHub inbox**: pending review requests, unread mentions and assignments, and assigned issues
  where someone else commented after you. Anything a workspace already handles stays on that
  workspace's card instead.
- **GitHub Projects boards** (optional): your items on a board, filtered like one of its views. Code
  review items join the inbox as "review owed" (or the card of your review worktree), the rest get a
  compact section per board grouped by status, and status moves show up in the briefing.
- **Remote machines**: workspaces on saved Herdr SSH machines are included, and a remote checkout of
  the same branch as a local workspace is shown on the local card (for example, experiments running on
  a build box for a paper you are writing locally).
- **Sidebar tokens** in Herdr: `✗ … ✓` for CI, `✎` new feedback, `↻` re-review, `queued` / `merged` / `closed`.
- **Notifications** once per new event: CI failing, new feedback, review requests, CI turning green.
- **A triage popup** (`prefix+t`): jump to a workspace, open its PR, park, snooze or dismiss items.
- **An orchestrator Claude session**, read-only: `/status` briefs you and explains the top items,
  `/briefing` fills in the morning briefing. It never touches your other sessions or repositories;
  it suggests prompts for them instead.
- **`TODO.md`** generated from the board, with a Notes section that is yours and never rewritten, and
  **daily briefings** in `briefings/` (weekdays at 09:00 or when you open the workspace, plus a
  "What moved today" section at 18:00).

## Requirements

- [Herdr](https://herdr.dev) 0.9.3 or newer, with its Claude Code integration (`herdr integration install claude`)
- Python 3.11 or newer (remote machines only need Python 3.9 for the read-only probe)
- [GitHub CLI](https://cli.github.com) logged in (`gh auth login`); for Projects boards also
  `gh auth refresh -s read:project`
- git; [fzf](https://github.com/junegunn/fzf) for the popup; ssh for remote machines
- [Claude Code](https://claude.com/claude-code) for the orchestrator session and briefings (the board
  itself works without it)

macOS and Linux. Nothing needs to be installed on remote machines.

## Install

Run these from a shell pane inside Herdr:

```sh
git clone https://github.com/Dtenwolde/herdr-orchestrator ~/git/orchestrator
cd ~/git/orchestrator
bin/orch-setup                    # checks requirements, creates state/overrides.toml
bin/orch-setup --config --launch  # adds the sidebar tokens and prefix+t to Herdr, opens the workspace
```

`--config` backs up `~/.config/herdr/config.toml`, appends two blocks, validates the result with
`herdr config check` and restores the backup if Herdr rejects it. If you already have your own
`[ui.sidebar.spaces]` rows or `prefix+t` is taken, it prints the snippet for you to add instead.
`--launch` opens a workspace called `orchestrator` with Claude on the left and the board on the right.

The first time Claude starts in this folder it asks whether you trust it; the folder pre-approves the
read-only commands the orchestrator uses (see `.claude/settings.json`). Then type `/status` there.

To set it up by hand instead: run `bin/orch-watch` in any Herdr pane, and see `bin/orch-setup` for the
two config blocks.

## Daily use

| | |
|---|---|
| Board | Watch the right-hand pane. Enter refetches everything; otherwise it refreshes on events and every minute. While you scroll up it holds still and catches up when you scroll back down. |
| `prefix+t` | Triage popup: enter jump, `ctrl-o` open link, `ctrl-p` park, `ctrl-s` snooze, `ctrl-d` done. |
| `/status` | In the orchestrator session: a short briefing on the top items with a concrete next step each. |
| `/briefing` | Fills in today's `briefings/YYYY-MM-DD.md` (the board asks for this itself on weekdays). |
| Tell it | "Park docs-refresh", "snooze #412 until Monday", "#678 is handled in flaky-tests", "I already did that". |

## Configuration

**`state/overrides.toml`** is how you steer the ranking, per workspace label or per PR/issue. The
popup and the orchestrator edit it for you, keeping your comments; see `overrides.example.toml`.

| Field | Effect |
|---|---|
| `parked = true` | Move it to PARKED, no notifications, until you remove it. |
| `snooze_until = 2026-11-10` | PARKED until that day starts (a datetime or ISO string works too). |
| `ignore_question = true` | Stop flagging "the agent ended with a question". |
| `note = "..."` | Shown on the board, in the popup and in `TODO.md`. |
| `links = ["owner/repo#n"]` | Issues or PRs this workspace handles; their inbox activity shows on its card. |
| `done = true` or a date | Dismiss an inbox item; with a date, newer activity brings it back. |

**`orchestrator.toml`** (optional, copy `orchestrator.example.toml`) holds settings: which branches
never have their own PR, extra bot logins to ignore, the briefing and end-of-day times, the refresh
interval, whether to restore `r`/`b`/`f` workspace kind tags, and which GitHub Projects boards to
read (`[[projects]]`: owner, number, and the view whose filter to use, plus the field that marks
code review items). `ORCH_INTERVAL`,
`ORCH_BRIEFING_AT`, `ORCH_WHAT_MOVED_AT` and `ORCH_AGENT` override the `[watch]` settings.

**`CLAUDE.local.md`** (optional, git-ignored) gives the orchestrator personal context: your projects,
machines and habits.

## How it works

`bin/herdr-status` builds the board. It reads Herdr (`herdr workspace/agent/pane list`, and
`herdr --machine` for remote machines), git in each checkout, the Claude Code session logs in
`~/.claude/projects` (last prompt, last reply, the session's own recap), and GitHub through `gh`.
Remote git state and session logs come from a small read-only probe sent over ssh. Every reason
becomes an item with a stable id such as `pr:acme/widgets#412:ci-failing`, which overrides,
notifications, `TODO.md` and the briefings key on.

GitHub results are cached in `state/cache.json`: 90 s while CI runs, 5 minutes for other open PRs,
an hour for merged ones, and a new commit invalidates the entry. A failed GitHub call keeps the
last known value. `bin/orch-watch` subscribes to Herdr's event socket and redraws from the cache
when something changes.

Useful flags: `--json` (everything, for scripts and the orchestrator), `--cached`, `--max-age 0`,
`--workspace <text>`, `--local`, `--todo`, `--briefing`, `--what-moved`.

Files it writes, all inside this folder: `state/` (cache, items, history, overrides), `TODO.md`,
`briefings/`. Outside it, it only writes display metadata to Herdr (sidebar tokens) and shows Herdr
notifications. It never writes to your repositories, your sessions or GitHub.

## Troubleshooting

- **The board says "polling (no Herdr events)"**: it could not subscribe to Herdr's socket; check that
  it runs inside Herdr (`HERDR_ENV=1`). It still refreshes every minute.
- **A remote machine shows a warning**: the ssh login probably expired; the board continues with the
  local data.
- **Sidebar tokens do not show**: they need the `[ui.sidebar.spaces]` row from `bin/orch-setup --config`.
- **An override "matches no current workspace or PR"**: the item went away on its own; delete the entry.
- **Something looks wrong**: `bin/herdr-status --json --workspace <name>` shows the raw data and the
  reasons behind the ranking.

## Uninstall

Close the `orchestrator` workspace, remove the two `# orchestrator:` blocks from Herdr's
`config.toml` (or restore its `config.toml.bak-orchestrator-*` backup), and delete this folder.

## License

MIT, see [LICENSE](LICENSE).
