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
Git worktree. To move a number from its demo folder to a project:

```sh
cd ~/vswitch-team/dev && vswitch-vn-unassign
cd <PROJECT FOLDER> && vswitch-vn-assign -label dev -state-dir ~/.vswitch/dev -trust "<REVIEWER'S VN>"
```

Then close the demo chat and start a new one in the project folder. The
number, its saved state and its notes carry over; do not enroll again. Assign
adds nothing to your project's shared files: its entries are local to you
(`~/.claude.json` and the folder's `.claude/settings.local.json`). Each agent
gets only its own phone state, never your account directory or another
agent's credential.

## Decide when they are available

An assigned chat listens while it is open: its background `vswitch-wait`
wakes it when a call or note arrives. A closed chat hears nothing, and the
next chat opened in that folder picks the number up. Tell the receiver what a
call is about before the caller dials, as in the README's step 5. A number
answers in one chat at a time, so open one chat per assigned folder.

A note works for a handoff such as "review this tomorrow." A live call works
for discussing a result. Sending either does not start a stopped agent.
Watch provider usage and leave enough capacity to checkpoint and report.

Before hanging up, send the final summary and wait for the peer to acknowledge
it in a message. Agree which party hangs up. A viewer showing your message,
or a local `vphone_ack` cursor update, is not proof the peer has read it.
After a switch restart the parties make a new call using their own context.

## Optional: let Claude Code hear the phone ring

Assigned chats already listen with `vswitch-wait`, which works on every
Claude login. If your Claude Code supports channels (a research preview), you
can have rings arrive as channel events instead. Reassign each folder with
`-ring channel`:

```sh
cd ~/vswitch-team/dev && vswitch-vn-unassign && \
  vswitch-vn-assign -label dev -state-dir ~/.vswitch/dev -ring channel -trust "<REVIEWER'S VN>"
```

Then start Claude in that folder with this command **instead of plain
`claude`**:

```sh
claude --dangerously-load-development-channels server:vswitch
```

If channels are unavailable in your Claude environment (Bedrock, Google
Cloud/Vertex AI, Microsoft Foundry, or a company account that blocks
channels), the chat is told to fall back to `vswitch-wait`. Peer messages
never grant permission to override your instructions. The channel is
specific to Claude Code. [PROMPTS.md](PROMPTS.md#let-your-agents-hear-the-phone-ring-claude-code)
has a checklist for when rings don't arrive.

## Optional: add Codex

Codex can be a teammate too, connected by hand as in beta.3.
`vswitch-vn-assign` sets up Claude Code only in this release. You, the account
holder, give Codex a separate number. The command below configures Codex
user-wide, so all sessions using that entry share the number; use one such
session for this setup:

```sh
vswitch-account enroll -dir ~/.vswitch/account -state-dir ~/.vswitch/codex -name Codex
codex mcp add vswitch -- "$(brew --prefix)/bin/vswitch-mcp" -state-dir "$HOME/.vswitch/codex"
```

Restart Codex, give it the other agent's number, and give that agent Codex's.
Codex has no background listener in this release, so it hears a call only
while you have told it to wait. When Codex is the receiver, paste this before
the caller dials:

```text
Use your vPhone through the vswitch MCP server. Read the vPhone skill at
vphone:///SKILL.md and resume your own number. Keep all credentials private;
do not read another agent's private phone state or the account holder's
private directory.

<CALLER> at <CALLER'S VN> is about to call about <TASK>. Check your
invitations with bounded waits for up to three minutes and answer that call.
Exchange and acknowledge the final summary in a call message before the call
ends, then tell me what you agreed. If no call arrives in three minutes, stop
and tell me. Do no other work.
```

When Codex is the caller, the README's Dev prompt works as written. Codex
does not use Claude's channel flag.
To let a Claude agent hear Codex in full:
`vswitch-vn-trust add "<CODEX'S VN>" -name Codex` in the Claude project folder.

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
notes, and `vswitch-edit` browses past calls. In a terminal, each call's
participants are colored: whoever started the call is white, and the others
get yellow, green, amber and so on in the order they join. Colors belong to
the call, not the number. Set `VSWITCH_COLOR=never` (or `NO_COLOR=1`) to turn
them off, or `VSWITCH_COLOR=always` when piping into `less -R` (it wins over
`NO_COLOR`). The viewers show your numbers as `Name (number)`: each assigned
number by its label, and any number by the name you gave it with
`vswitch-vn-trust add <VN> -name <NAME>`.

Ask each agent to report what changed, what it actually verified and what
remains. The installed `share/vswitch/skill/prompts/cooperate.md` provides
more agent-facing guidance for shared coding work; the README remains your
human setup checklist.
