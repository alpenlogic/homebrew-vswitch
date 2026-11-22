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
next chat opened in that folder picks the number up. Start chats with
`vswitch-claude` (or `vswitch-claude -c` to continue the last one): the chat
starts its listener at once. A plain `claude` chat waits until you type
before it starts listening. Tell the receiver what a
call is about before the caller dials, as in the README's step 5. A number
answers in one chat at a time, so open one chat per assigned folder.

An agent busy with a turn, or already on a call, can't pick up. That's
expected:

- A call rings for 30 seconds, then the switch ends it as *no answer*. The
  caller is told and can leave a note. Set a different time for a number with
  `vswitch-vn-assign -ring-timeout 20s` when you assign it (5 seconds to 5
  minutes); to change it later, edit `ringTimeoutSeconds` in the number's
  `settings.json`.
- Calling a number that is on another call is refused at once as *busy*, and
  the caller can leave a note.
- Either way, the person called gets a missed-call record, and an assigned
  Claude Code chat is told about it, and about any notes, at the start of its
  next turn.
- After the computer sleeps, listeners reconnect and tell the agent about the
  gap, so it checks what its teammates did meanwhile.

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

Then start Claude in that folder with `vswitch-claude`, which loads the
channel for you. With plain `claude`, add the flag yourself, every time:

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

Codex can be a teammate too. You, the account holder, give it its own number
and its own folder:

```sh
vswitch-account enroll -dir ~/.vswitch/account -state-dir ~/.vswitch/codex -name Codex
mkdir -p ~/vswitch-team/codex
cd ~/vswitch-team/codex && vswitch-vn-assign -app codex -label codex -state-dir ~/.vswitch/codex -trust "<DEV'S VN>"
```

Review the project and its hooks in Codex (`/hooks`); setup never grants
trust for you. Then start Codex in that folder with:

```sh
cd ~/vswitch-team/codex && vswitch-codex
```

`vswitch-codex` supports **Codex CLI 0.167.1** and refuses other versions.
It starts Codex together with a private helper so calls and notes wake the
chat, and each turn starts by checking the inbox. Closing that Codex window
stops the helper; nothing keeps running in the background. Pass other Codex
arguments after `--`, for example `vswitch-codex -- resume <THREAD ID>`.
This launch is new in beta.5. Its parts are tested, and Codex's wake-up was
shown working in our call tests, but the one-command launch had not yet been
run end to end on a live chat when this guide was written. Please report
anything odd with `vswitch-doctor`.

Codex may ask its approval reviewer before answering a call. It doesn't need
to: answering is pre-approved for numbers on the trust list
(`vswitch-vn-trust`), and `vphone_answer` itself refuses any other caller
unless the call includes `untrusted: true`. If Codex lets you allow a tool
only when its arguments match, allow `vphone_answer` calls that don't set
`untrusted`. If it can only allow a tool outright, keep the reviewer: an
outright allow would also cover untrusted calls.

Started as plain `codex`, the folder is configured but Codex gets no vSwitch
tools or wake-ups. In that case, or with another Codex version, connect it by
hand as in beta.3 instead of assigning the folder. This configures Codex
**user-wide**, so every Codex session using that entry shares the number;
use only one such session at a time:

```sh
codex mcp add vswitch -- "$(brew --prefix)/bin/vswitch-mcp" -state-dir "$HOME/.vswitch/codex"
```

Such a Codex hears a call only while you have told it to wait. When it is the
receiver, paste this before the caller dials:

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

Either way, give Codex the other agent's number and give that agent Codex's.
When Codex is the caller, the README's Dev prompt works as written. Codex
does not use Claude's channel flag.
To let a Claude agent hear Codex in full:
`vswitch-vn-trust add "<CODEX'S VN>" -name Codex` in the Claude project folder.

## Add a third participant when you need one

Start with a pair before adding more roles. A participant already on a call
can invite another number; a third agent cannot join itself from outside.
Ask the inviter to explain the task and choose what retained history to share.
By default a newly added participant starts at its connection point.

Each VN's active-call allowance defaults to one. An agent busy on another call may need to finish
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
