# Build your team after the first call

Start with the numbered **[human walkthrough in README.md](README.md)**.
It takes you from installation to two Claude agents completing a task, with
commands, prompts and expected results. This page covers what to try next.
Reuse the account, numbers and private directories you already created.

## Pick a useful division of work

Give the team one shared goal and complementary roles:

| Goal | First agent | Second agent |
| --- | --- | --- |
| Improve a feature | proposes and implements a change | reviews and tests it |
| Write something | drafts it for the intended audience | checks clarity and omissions |
| Investigate a bug | reproduces and narrows the problem | independently checks the explanation |
| Plan a project | proposes steps and tradeoffs | challenges assumptions and risks |

Tell them when to involve you: a design decision, a disagreement, an install,
a push or a deployment. Have them agree who edits each file before they begin.
Keep their messages short; exchange paths or commit references for large work.

For an existing code project, each agent should use its own working copy or
Git worktree. Close its demo Claude window, add its folder-local MCP connection
inside the chosen project folder, and restart it there. Keep the same number
and private `-state-dir`; do not enroll again or run two sessions against that
same state directory. Each agent gets only its own phone state, never your
account directory or another agent's credential.

## Decide when they are available

Without a ring channel, start the receiver waiting before each call. Paste a
bounded waiting prompt into its conversation first, then give the caller its
task. Merely keeping an idle window open does not make the model check calls.
The README's step 5 shows both prompts in the correct order.

A note works for a handoff such as "review this tomorrow." A live call works
for discussing a result. Sending either does not start a stopped agent.
Watch provider usage and leave enough capacity to checkpoint and report.

Before hanging up, send the final summary and wait for the peer to acknowledge
it in a message. Agree which party hangs up. A viewer showing your message,
or a local `vphone_ack` cursor update, is not proof the peer has read it.
After a switch restart the parties make a new call using their own context.

## Optional: let Claude Code hear the phone ring

Once the first call works, you can enable Claude Code's ring channel (a
research preview) in each agent's MCP connection. This lets new activity wake
an idle **running** session. It does not start Claude or answer on its behalf.

In your setup Terminal, replace both folder-local connections. Fill in the
same Dev/Reviewer numbers you used in the README:

```sh
cd ~/vswitch-team/dev && claude mcp remove vswitch -s local
cd ~/vswitch-team/dev && claude mcp add vswitch -s local -- "$(brew --prefix)/bin/vswitch-mcp" \
  -state-dir "$HOME/.vswitch/dev" -channel -channel-trust "<REVIEWER'S VN>"

cd ~/vswitch-team/reviewer && claude mcp remove vswitch -s local
cd ~/vswitch-team/reviewer && claude mcp add vswitch -s local -- "$(brew --prefix)/bin/vswitch-mcp" \
  -state-dir "$HOME/.vswitch/reviewer" -channel -channel-trust "<DEV'S VN>"
```

Restart both Claude windows, each in its own folder, using this command
**instead of plain `claude`**:

```sh
claude --dangerously-load-development-channels server:vswitch
```

Keep those sessions open. Names in `-channel-trust` are comma-separated VNs:
their notes arrive with the text; other numbers ring without their note text.
Peer messages never grant permission to override your instructions. If the
preview is unavailable in your Claude environment, use the receiver-wait
prompts instead. Other clients choose their own listening or notification
mechanism; this channel flag is specific to Claude Code.

## Optional: add Codex

Codex can be a teammate too. You, the account holder, give it a separate
number. The command below configures Codex user-wide, so all sessions using
that entry share the number; use one such session for this setup:

```sh
vswitch-account enroll -dir ~/.vswitch/account -state-dir ~/.vswitch/codex -name Codex
codex mcp add vswitch -- "$(brew --prefix)/bin/vswitch-mcp" -state-dir "$HOME/.vswitch/codex"
```

Restart Codex, give it the other agent's number, and give that agent Codex's.
Reuse the README's receiver/caller prompts with the appropriate roles and VNs.
One receives while the other dials. Codex does not use Claude's channel flag.

## Add a third participant when you need one

Start with a pair before adding more roles. A participant already on a call
can invite another number; a third agent cannot join itself from outside.
Ask the inviter to explain the task and choose what retained history to share.
By default a newly added participant starts at its connection point.

Your license limits the number of registered VNs, and each VN's active-call
allowance defaults to one. An agent busy on another call may need to finish
that call or arrange a later callback. A short note can coordinate that.

## Watch and review the work

`vswitch-tail` follows recent events, `vswitch-watch --all` follows calls and
notes, and `vswitch-edit` browses past calls. To display names, create
`~/.vswitch/names.json` with the first eight characters after each VN's colon,
for example `{"1b2c3d4e":"Dev","9f8e7d6c":"Reviewer"}`.

Ask each agent to report what changed, what it actually verified and what
remains. The installed `share/vswitch/skill/prompts/cooperate.md` provides
more agent-facing guidance for shared coding work; the README remains your
human setup checklist.
