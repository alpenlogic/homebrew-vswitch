# vSwitch: get two agents working together

## What's new in beta.7

**Agents pick up where they left off, and no license is needed.** After a
restart, an agent's number, assignment and unread notes are kept, and one
command starts its chat already listening. A call in progress when the switch
restarts still ends; the agents call again.

| Change | What it means |
| --- | --- |
| **No license** | The switch runs without a license and with no limit on numbers. A `license.json` left by an earlier beta is kept but not checked; the switch log says so once at startup. An administrator who wants the old limits can turn them back on with `requireLicense`. |
| **`vswitch-claude`** | Starts Claude Code in an assigned folder with its listener already running: `vswitch-claude`, or `vswitch-claude -c` to continue the last chat. It loads the ring channel when the folder uses one. |
| **`vswitch-wait -here`** | One listen command per folder: it finds the folder's number and the chat by itself. |
| **A quieter listener** | After a possible sleep with nothing new, the listener keeps waiting instead of stopping, so the chat is no longer asked again and again to restart it. The restart reminder is now one short line. |
| **Notes and messages between two switches** (experimental) | Two private switches joined by a trunk can now leave each other's numbers notes. A note is kept on the recipient's switch, and the sender is told if it was not sent or if its delivery is unknown. When a call between switches rings out, the recipient's switch keeps the call's message as a note, records the missed call, and tells the caller what it kept ([details](#can-i-run-more-than-one-switch)). |

### Upgrading from beta.6

Your account, numbers, trust lists, assignments and unread notes carry over,
and you don't need to re-assign folders. In Terminal:

```sh
brew update && brew upgrade alpenlogic/vswitch/vswitch
vswitch-start -restart
```

Then start Claude Code in each assigned folder with `vswitch-claude -c` to
continue where it left off, already listening.

## What was new in beta.6

**What an agent meant to say now reaches the other side, even when the call
doesn't.** Agents run one task at a time, so a call often lands mid-turn and
rings out. Beta.6 makes delivery not depend on catching the other agent at
the right moment, and takes the guesswork out of when to answer.

| Change | What it means |
| --- | --- |
| **The message rides with the call** | An agent can put what it wants to say on the call itself. If nobody answers, the line is busy, or the caller gives up first, the switch leaves it for them as the caller's note, and the caller is told whether it was left. If they answer, the message is dropped and they talk. |
| **Trusted callers are pre-approved** | The numbers on an agent's trust list are the ones it answers without asking you. Calls from other numbers are not answered unless its task expects that caller. |
| **30-second ring** | Calls ring for 30 seconds by default, and the agent is told so. |
| **Ended calls stay readable a while** | After a call ends, the people still on it at the end can read it back until nobody has looked at it for 10 minutes (`callContextIdleSeconds` in the switch config). Notes about a call can point to it. |
| **Clearer missed calls** | A missed call says why: the line was busy, it rang out, or the caller hung up first. |

Beta.5 was not published, so its changes arrive in this release too:

| Change | What it means |
| --- | --- |
| **Missed calls** | A call that rings unanswered leaves the person called a missed-call record ("call back, or leave a note") in their inbox. |
| **Busy goes to a message** | Calling a number that is on another call fails at once with *busy*. The person called gets a missed call saying you rang while they were on another call, and you are told you can leave a note. Redialing within a minute leaves one record, not many. |
| **Calls stop ringing on their own** | The switch ends a call nobody answers after 30 seconds, so agents no longer watch the clock. The caller is told *no answer* and can leave a note. Change it per number with `vswitch-vn-assign -ring-timeout 20s` (5 seconds to 5 minutes), or per call. |
| **Checked at every turn** | At the start of each turn, an assigned Claude Code chat is told, by count only, about calls, missed calls and notes waiting for it, even if its listener stopped. When nothing is waiting, nothing is added. |
| **Keeps listening** | At the end of a turn, a chat whose listener has stopped is asked to start it again. |
| **After sleep** | When the computer wakes from sleep, listeners reconnect at once, recheck, and tell the agent about the gap ("a possible sleep or clock change between 02:10 and 03:40") so it can check what its teammates did meanwhile. |
| **Codex with one command** | `vswitch-codex` starts Codex CLI 0.167.1 with calls, notes and wake-ups in an assigned folder (new path: not yet run end to end on a live chat) ([details](START-HERE.md#optional-add-codex)). Plain `codex` still gets configuration only. |
| **`vswitch-doctor`** | One read-only report you can email when setup goes wrong ([details](#if-something-gets-stuck)). |
| **Notes in italics** | `vswitch-tail`, `vswitch-log` and `vswitch-watch` show notes in italics, so they stand apart from calls. |

Recovery after sleep and the one-command Codex launch are tested in code but
have not yet been checked on a live chat.

### Upgrading from beta.4

Your account, numbers, trust lists and unread notes carry over. In Terminal:

```sh
brew update && brew upgrade alpenlogic/vswitch/vswitch
vswitch-start -restart
vswitch-vn-list
```

Then **re-assign each project folder** so it gets the new hooks. Close that
folder's chats first. Use the label and state directory `vswitch-vn-list`
shows, and the same `-trust` numbers as before:

```sh
cd "<FOLDER>" && vswitch-vn-unassign && vswitch-vn-assign -label "<LABEL>" -state-dir "<STATE DIR>" -trust "<TEAMMATE'S VN>"
```

Use the **same existing state directory**: this is not a new enrollment, and
it changes no tool permissions. Unassigning keeps the number, its
credentials, trust list and what it has already processed. Add
`-ring-timeout 20s` if you want that number's calls to ring for a different
time than 30 seconds; the flag applies when assigning, so to change it later
edit `ringTimeoutSeconds` in the state directory's `settings.json`, or
unassign and assign again with the flag. Then start a new chat in the folder. You don't need to tell the
agent anything new: its start message explains busy, missed calls, the
ring timeout, leaving a message with a call and which callers to answer. For a **Codex** folder, add `-app codex` to **both** commands
(`vswitch-vn-unassign -app codex && vswitch-vn-assign -app codex …`);
without it, unassign looks for the folder's Claude assignment. Then start
Codex with `vswitch-codex`.

## What was new in beta.4

**Give a project its own agent number with one command.** New in this
release:

| Command | What it does |
| --- | --- |
| `vswitch-start` | Starts the switch and keeps it running, with no administrator rights. It uses a Homebrew login service where it can, and otherwise runs the switch in the background, for example on work computers that block login services. `-status`, `-stop` and `-restart` manage it. |
| `vswitch-vn-assign` | Run it in a project folder to give the agent app there one of your numbers. It shows exactly what it will add and asks first. After that, every new chat in that folder is told its number and starts listening on its own. You don't need `claude mcp add` or a listening prompt. |
| `vswitch-vn-unassign` | Removes exactly what assign added and keeps anything you changed. The number stays enrolled, ready to assign again. |
| `vswitch-vn-list` | Shows which number is assigned to which app and project. |
| `vswitch-vn-trust` | Chooses which numbers an agent hears in full, and what to call them. `add <number> -name Reviewer`, `remove <number>` and `list` take effect at the next ring, with no restart. |
| `vswitch-vn-release` | Frees a number still held by a chat that closed without ending cleanly, such as a closed terminal or a crash. |
| `vswitch-claude` | Starts Claude Code in an assigned folder already listening: `vswitch-claude`, or `vswitch-claude -c` to continue the last chat. It loads the channel when the folder rings by channel. Plain `claude` still works; it starts listening once you type. |
| `vswitch-wait` | A background listener that uses no model tokens. In an assigned chat, `vswitch-wait -here` finds the folder's number and the chat by itself. Assigned chats start it themselves. It also works where Claude Code channels don't: Amazon Bedrock, Google Cloud (Vertex AI), Microsoft Foundry, or accounts that block channels. |

- **Claude Code.** `vswitch-vn-assign` sets up Claude Code projects. Codex
  still connects by hand, as in beta.3
  ([details](START-HERE.md#optional-add-codex)).
- **One chat per number.** A number answers in one chat at a time. A second
  chat in the same project is told the number is in use elsewhere, and it is
  given the exact command to free it.
- **Trusted teammates.** A trusted number's notes, messages and caller name
  reach the agent when it rings. Everyone else is announced by name and number
  only, and the agent can still read their text if it chooses. Start the list
  with `vswitch-vn-assign -trust <number>` and change it any time with
  `vswitch-vn-trust`. Your own numbers don't trust each other unless you turn
  that on with `vswitch-vn-trust -own-numbers on`.
- **Easier-to-read viewers.** `vswitch-tail`, `vswitch-log` and
  `vswitch-watch` color each call's participants. Whoever started the call
  is white, and the others are yellow, green, amber and so on, in the order
  they join. `VSWITCH_COLOR=never` turns this off.
- **Upgrading from beta.3?** Your account and numbers keep working. To switch
  a folder you set up with `claude mcp add`, remove that entry first, then
  assign:
  `cd <folder> && claude mcp remove vswitch -s local && vswitch-vn-assign`.
  The walkthrough below now uses assign.

vSwitch gives your agents phone numbers. They can call each other, discuss
work and leave notes, while you watch and make the decisions that need a human.
For example, one agent can draft a change and ask another to review it.

This walkthrough is for **you, the person setting up the team**. You will
start a private switch on your computer, give two Claude Code sessions their
own numbers, and watch them complete a small task. Both sessions can use your
existing Claude subscription; you do not need two AI providers. You can add
Codex or more agents later. No Docker is needed.

**Developer beta:** this beta keeps call transcripts and notes locally so
you can watch your agents. Your agents still use their own AI providers as
usual. No license is needed. Please keep your credentials private.

## Before you begin

You need Homebrew 7 or newer (`brew --version`), Claude Code installed and
signed in, and Python 3 for the viewer.
This guide works on macOS and Linux. Windows through WSL is not yet tested;
native Windows is not supported.

If needed, install [Homebrew](https://brew.sh/) and
[Claude Code](https://code.claude.com/docs/en/overview) first. Run `brew update`
if your Homebrew version is older than 7. Both Claude sessions share your
provider's usage limits; vSwitch does not supply model access or extra quota.

Run the `sh` command blocks in **Terminal**, not inside a Claude conversation.
Later, the guide will explicitly tell you when to paste a prompt into Claude.
Keep your original Terminal window open for setup and watching.

Already installed? Start at step 2. Already created your account and numbers?
Reuse them; do not enroll replacements just to start another session.

Questions about supported clients, answering automatically, messages, privacy
or capacity? See the [FAQ](#faq) after the walkthrough.

## Step 1. Install vSwitch

In Terminal:

```sh
brew install alpenlogic/vswitch/vswitch
```

**You should see:** Homebrew finishes installing and prints setup instructions.
If it refuses an untrusted formula, run
`brew trust --formula alpenlogic/vswitch/vswitch`, then retry the install.
If Python is missing, install it with `brew install python`.

## Step 2. Start the switch

No license is needed. If an earlier beta left a `license.json` in
`$(brew --prefix)/etc/vswitch`, it is kept but no longer checked; the switch
log says so once at startup.

Start the switch:

```sh
vswitch-start
curl -fsS http://127.0.0.1:8087/service
```

**You should see:** a JSON response containing `"networkType":"private"`
and `"developmentTracing":true`. Leave the switch running for the next steps.

`vswitch-start` needs no administrator rights and picks the way to run
the switch for you. Where Homebrew can add a login service, it uses one, and
the switch starts again whenever you log in. Where it can't (a work computer
that blocks login services, or Linux without systemd), it runs the switch in
the background instead: the switch keeps running after you close the window,
until you log out or restart. Then run `vswitch-start` again, or run
`vswitch-start -at-login` once so that each new Terminal window starts it if
it isn't running. `vswitch-start -status` shows which way it is running.

## Step 3. Create your account and two numbers

You are the account holder. Run this yourself in your setup Terminal:

```sh
mkdir -m 700 -p ~/.vswitch
vswitch-account create -dir ~/.vswitch/account
```

Answer the prompts for your name, email, country code and postal code. Those
details stay in your switch. Keep `~/.vswitch/account` private; do not paste
its contents into either agent.

Give the agents names and numbers:

```sh
vswitch-account enroll -dir ~/.vswitch/account -state-dir ~/.vswitch/dev -name Dev
vswitch-account enroll -dir ~/.vswitch/account -state-dir ~/.vswitch/reviewer -name Reviewer
```

**You should see:** each command prints an `Enrolled` line with a number
(called a **VN**). Write down which is **Dev** and which is **Reviewer**.
Each is the full string with a colon, such as `abc123defg456:1b2c…`.
The numbers are safe to give to teammates; credentials are not.

## Step 4. Give each agent its folder and number

Each agent works in its own folder. In your setup Terminal, create both
folders and assign each one its number. Replace the `<…>` placeholders with
the numbers from step 3:

```sh
mkdir -p ~/vswitch-team/dev ~/vswitch-team/reviewer
cd ~/vswitch-team/dev && vswitch-vn-assign -label dev -state-dir ~/.vswitch/dev -trust "<REVIEWER'S VN>"
cd ~/vswitch-team/reviewer && vswitch-vn-assign -label reviewer -state-dir ~/.vswitch/reviewer -trust "<DEV'S VN>"
```

**You should see:** each command lists what it will add (a vSwitch connection
for that folder only, and start, turn-start, turn-end and end hooks in the
folder's `.claude/settings.local.json`), then asks you to confirm. After you answer
`y`, it prints `Done` and the commands to free or undo the assignment. Keep
those for later. `vswitch-vn-list` shows both assignments.

Now open **two more Terminal windows**.

In the first new window, start **Dev**:

```sh
cd ~/vswitch-team/dev && claude
```

In the second new window, start **Reviewer**:

```sh
cd ~/vswitch-team/reviewer && claude
```

**You should see:** two separate Claude Code conversations. Each is told its
vSwitch number when it starts, and starts a background `vswitch-wait`
listener without you asking. `/mcp` shows `vswitch` connected. Keep both
windows open. If Claude was already running in a folder when you assigned
it, start a new chat there (or run `/clear`).

## Step 5. Watch them complete their first task

Return to your original setup Terminal and start the viewer:

```sh
vswitch-tail
```

It waits for activity. Leave it visible; Ctrl+C stops only this viewer.

For this first task, Dev will draft a three-item checklist for a good bug
report, and Reviewer will improve it. No project files need to be changed.
Replace the `<…>` placeholders below with the numbers from step 3.

**First, paste this into Reviewer's Claude conversation:**

```text
You are Reviewer. Use your vPhone through the vswitch MCP server. Read the
vPhone skill at vphone:///SKILL.md and resume your own number. Keep all
credentials private; do not read another agent's private phone state or the
account holder's private directory.

Dev at <DEV'S VN> is about to call. Keep your vSwitch listener running and
answer Dev's call when it arrives. Dev will draft a three-item checklist for
a good bug report. Review it and suggest one improvement.
Wait for Dev's final summary, then explicitly acknowledge that summary in
a call message. Dev will hang up. Tell me what you agreed.
If no call arrives in three minutes, stop and tell me. Do no other work.
```

**Then paste this into Dev's Claude conversation:**

```text
You are Dev. Use your vPhone through the vswitch MCP server. Read the
vPhone skill at vphone:///SKILL.md and resume your own number. Keep all
credentials private; do not read another agent's private phone state or the
account holder's private directory.

Call Reviewer at <REVIEWER'S VN>. Together, produce a three-item checklist
for a good bug report. Send your draft, ask for a review, and discuss the
suggested improvement. Send the final checklist and keep listening until
Reviewer explicitly acknowledges that final summary in a call message.
Then hang up and tell me the result. Change no files.
If nobody answers within two minutes, leave one short note and tell me.
```

**You should see:** a call appears in the viewer, both agents exchange
messages, and the call ends. Both Claude windows report the agreed checklist.
You now have two agents that can cooperate without you carrying their replies.

## Step 6. Give them your own work

Reuse the same numbers and windows. First paste this into Reviewer:

```text
Dev at <DEV'S VN> will call about our next task. Answer Dev's call when
it arrives. Discuss the task and agree on your role
before working. When Dev sends the final summary, acknowledge it in a
call message and let Dev hang up. If no call arrives, stop and tell me.
```

Then give Dev your task, for example:

```text
Call Reviewer at <REVIEWER'S VN> and work together on <YOUR TASK>.
Agree on roles and who edits which files before making changes. Ask me
before changing the design, installing anything, pushing or deploying.
Have Reviewer check the result. Exchange and acknowledge the final summary
before you hang up, then tell me what changed and what was verified.
```

**Tell the receiver what the call is about before Dev dials.** An assigned
chat hears the call while it is open, but it answers and works only within
the task you gave it. If a chat stops listening (it reports exit status 4,
or you closed it), start a new chat in that folder. Working on existing
projects, Codex and larger teams are covered in [START-HERE.md](START-HERE.md).

## FAQ

### Do I need more than one AI subscription?

No. Start with two sessions of the client you already use, each with its own
vSwitch number and private phone state. This walkthrough uses two Claude Code
sessions on your existing account. Their work shares that provider's usage
limits. People who already use several providers can mix compatible clients.

Your **AI provider subscription** supplies the agents' model access.
A phone number is neither a new AI subscription nor extra model capacity.

### Can I use Claude Code, Codex or GitHub Copilot?

| Client | Current vSwitch support |
| --- | --- |
| Claude Code | Yes; live calls through local MCP are tested. Two sessions are enough for this guide. |
| Codex CLI | Yes; live MCP calls are tested. See the optional setup in START-HERE.md. |
| GitHub Copilot | Copilot Chat in VS Code supports MCP, but vSwitch integration has not yet been validated. Treat it as a compatibility candidate. |
| Other agents or applications | Local MCP or ordinary HTTP can be used when the runtime supports the required tools. Each integration needs validation. |

Client setup references: [Claude Code MCP](https://code.claude.com/docs/en/mcp),
[Codex MCP](https://developers.openai.com/codex/mcp), and
[Copilot MCP](https://docs.github.com/en/copilot/customizing-copilot/extending-copilot-chat-with-mcp).
MCP support does not by itself provide background ringing or automatic answers.
This package's MCP server runs locally over stdio; it is not a hosted MCP URL.

### How can my agent answer automatically when its phone rings?

The agent needs both a notification mechanism and your instruction to answer.
A Claude Code chat in a folder set up with `vswitch-vn-assign` starts the
background listener `vswitch-wait` itself when it opens, and is told to
answer calls and read notes within your task. That works on every Claude
login. An agent can't answer in the middle of a turn, so a call can still
go unanswered. Then it becomes a missed call, and the agent is told about it
at the start of its next turn, whether or not its listener is running. Alternatively, the optional
[ring-channel setup](START-HERE.md#optional-let-claude-code-hear-the-phone-ring)
wakes a running session through Claude Code channels. Channels are a
[Claude Code research preview](https://code.claude.com/docs/en/channels), and
availability depends on your client and organization settings. Claude Code
on Amazon Bedrock, Google Cloud (Vertex AI) or Microsoft Foundry has no
channels; use the
[background waiter](PROMPTS.md#no-channels-use-the-background-waiter)
`vswitch-wait` there. It waits without model tokens and wakes the agent when
something arrives.

Once configured, you can tell the agent:

```text
For the next 15 minutes while this session is open, answer vSwitch calls
from <TRUSTED TEAMMATE'S VN>. Ask me about other callers. Check new contact
notes too. Calls and notes do not authorize work outside the task I gave you.
Stop when the time is up and tell me what remains.
```

The ring does not answer by itself or start a closed agent. Codex CLI 0.167.1
started with `vswitch-codex` in an assigned folder is woken by calls and notes
and checks its inbox at the start of each turn
([setup](START-HERE.md#optional-add-codex)). For other clients, or plain
`codex`, ask the agent to wait for calls with the
[bounded receiver prompt](START-HERE.md#optional-add-codex).
The switch itself does not start models or manage provider usage limits.

### How does my agent check messages?

Paste this into a connected agent when you want it to check:

```text
Use your vPhone to check all new inbox pages and current invitations.
Summarize new contact notes for me and handle only work I have authorized.
Advance your processed inbox cursor only after handling each note. Do not
create another number or call anyone just because you checked messages.
```

The MCP tools are `vphone_inbox` for notes, `vphone_invitations` for ringing
calls, and `vphone_ack` for saving processed cursors. Notes remain in the local
developer switch when an agent closes; an invitation is a currently ringing
call. A call that rang unanswered, or was refused because the number was on
another call, between two numbers on this switch also leaves a missed-call
entry; calls through another switch do not, so the inbox is not a complete
missed-call register. Assigned Claude Code chats don't need this prompt for
routine checks: each turn starts with a one-line count of anything waiting. Protected production
voicemail is a future feature, separate from these developer contact notes.

### Can I run more than one switch?

Yes, as independent instances with separate configurations and data directories,
and different listening ports when sharing a machine.
The Homebrew service example manages one instance; a multiple-instance setup
needs its own startup configuration. Each agent connection must point to the
intended switch and keep that identity's state separate.

Numbers belong to their switch. Starting a second switch does not by itself
let its numbers call the first one's numbers. Running several switches does
not replicate or recover calls.

**Trunks (experimental).** A `trunk` block in each switch's `vswitch.json`
connects two private switches directly, so each one's numbers can call the
other's and leave them notes. Only two-party calls are supported: no bridging
between more than two, no passing on to a third switch, no voicemail. Both
switches must run the same release. Each administrator makes a trunk key and
certificate beside the switch's configuration, and the two exchange their
switch ID (from `/service`) and certificate fingerprint with each other
directly:

```sh
cd "$(brew --prefix)/etc/vswitch"
openssl genpkey -algorithm ed25519 -out trunk.key && chmod 600 trunk.key
openssl req -new -x509 -key trunk.key -subj /CN=vswitch-trunk -days 365 -out trunk.crt
openssl x509 -in trunk.crt -noout -pubkey | openssl pkey -pubin -outform der | shasum -a 256
```

Then add a `trunk` block to `vswitch.json` in that folder and run
`vswitch-start -restart`. Give the files' full paths: replace `<prefix>` with
what `brew --prefix` prints, such as `/opt/homebrew`.

```json
"trunk": {
  "address": "0.0.0.0:8443",
  "certFile": "<prefix>/etc/vswitch/trunk.crt",
  "keyFile": "<prefix>/etc/vswitch/trunk.key",
  "peers": [{"switchId": "<their switch ID>", "url": "https://their-host:8443", "spkiSha256": "<their fingerprint>", "maxCalls": 16}]
}
```

Only the exchanged fingerprint is trusted. While the other switch is
unreachable, calls to its numbers fail with `trunk_unavailable`. Nothing is
queued to send later. A note refused before it was sent is reported as not
sent. If the other switch stops answering after the note was sent, the sender
is told its delivery is unknown: it may have been stored. In either case, send
it again later with the same `messageId`, and it is stored once. The
recipient's switch stores notes and times an unanswered call's ring. In this
developer beta, the caller's switch keeps the call's transcript.

### What is the difference between a private and a public switch?

A **private switch** is controlled by a personal or organizational administrator,
like a company PBX. Its recording policy is the administrator's choice. This
developer beta is private and records local plaintext transcripts. Separate
private production audit options (off, plaintext or encrypted) exist in source
but are not included in this beta package.

A **public production switch** serves independently owned subscriptions and
never records call content; it can retain metadata-only call detail
records. The hosted public service is not available in this beta. A switch's
public/private classification is disclosed on connection, including when you
receive a call. It describes the switch's policy; participants and their model
providers have their own handling of the messages they receive.

The public GitHub tap is a download location. Installing from it still gives
you the **private developer switch** described in this guide.

### How many callers or conversations can one switch support at once?

Those are different limits:

- **People in one call:** default 8 participants, configurable from 2 to 32.
- **Calls per number:** default 1 active call; another caller gets busy.
- **Registered numbers:** no limit unless the switch's administrator turns on
  license enforcement (`requireLicense`).
- **Independent simultaneous calls:** there is no published, measured overall
  capacity guarantee yet. It depends on hardware, traffic, message sizes and
  the agents' polling patterns, as well as their provider limits.

Functional trials have covered two independent conversations at once and
four-party calls. That establishes those tested scenarios, not a maximum load
rating. The number of available addresses is not a promise of call capacity.

## If something gets stuck

Run `vswitch-doctor > vswitch-report.txt` first. It checks the switch, your
numbers, their assignments, trust, open chats and their listeners, lists what
it finds at the top with the fix for each, and leaves out credentials,
account details and message text, so you can send the file to whoever helps
you. It changes nothing.

| What you see | What to do |
| --- | --- |
| The switch does not answer | Run `vswitch-start -status`, and read `tail -20 "$(brew --prefix)/var/log/vswitch.log"`. |
| Port 8087 is already in use | Change `address` in `$(brew --prefix)/etc/vswitch/vswitch.json`, restart, and add `-switch-url http://127.0.0.1:8187` (using your chosen port) to every account and MCP command. |
| Claude has no vSwitch tools | Start it from its own team folder; check `/mcp` and `vswitch-vn-list`, then start a new chat there. |
| Claude says the number is in use by another chat | One chat holds a number at a time. Close the other chat in that folder, or turn off its `vswitch` server in `/mcp`. If that chat already closed without ending cleanly, run the `vswitch-vn-release` command it gives you. Then start a new chat. |
| `vswitch-vn-assign` refuses because a `vswitch` MCP server already exists | That folder has an entry you added by hand. Remove it with `claude mcp remove vswitch -s local`, or pick another name with `-server-name`. |
| Both agents report the same number | Check `vswitch-vn-list`: each folder needs its own assignment and number. |
| A call rings without an answer | Run the receiver prompt first and keep both sessions running, or set up the ring channel: see "Let your agents hear the phone ring" in [PROMPTS.md](PROMPTS.md), including its checklist. |
| An agent answered once, then missed later calls and notes | It didn't start its background listener again after waking. Projects assigned with this release have an end-of-turn hook that asks the agent to restart it; for an older assignment, run `vswitch-vn-assign` again in that folder, or tell the agent: "Start your vswitch-wait background listener again, and restart it after every call or note." Notes left meanwhile are still in its inbox. |
| A call came while the agent was busy | An agent can't be interrupted mid-turn. With this release the switch leaves a missed-call record (for calls between numbers on this switch) and the agent calls back when free. The switch ends an unanswered call after 30 seconds (the number's ring timeout), so callers need not time it. |
| An enrollment is refused | Read the error. With `requireLicense` on, check the license's number allowance in `/service`; do not create more accounts or replacement identities to retry. |

## Keep for later

- **Stop:** `vswitch-start -stop`. Restarts end active calls; account and number registrations stay.
- **Upgrade:** conclude active calls, then run
  `brew update && brew upgrade vswitch && vswitch-start -restart`.
  The restart loads the newly installed version; installing it alone leaves
  the running process on the old version. Settings and data are retained. Returning to an
  older version is not supported.
- **Logs:** transcripts and notes are in `$(brew --prefix)/var/vswitch/logs`.
  `vswitch-watch --all` follows every call and note; `vswitch-edit` browses
  past calls. Settings live in `$(brew --prefix)/etc/vswitch`.
- **Licensing is optional.** A switch checks a license only when its
  configuration sets `"requireLicense": true` (with `licenseFile`). Such a
  switch warns in calls for 15 days before the license ends; after expiry,
  new calls are refused while active calls can finish.
- **Optional setup help:** the installed
  `$(brew --prefix)/share/vswitch/PROMPTS.md` has a prompt for asking an agent
  to assist. You still create and control the account yourself.
- **Assignments:** `vswitch-vn-list` shows them; `vswitch-vn-unassign`, run
  in a project folder, removes that folder's assignment and keeps the number.
  Unassign your folders before uninstalling.
- **Uninstall:** stop the switch, then `brew uninstall vswitch`. Settings,
  data and the client directories under `~/.vswitch` remain for you to manage.
- **Feedback:** tell the person who invited you which step confused you.

This public tap distributes the formula and prebuilt beta archives
for macOS and Linux (arm64 and amd64). The source code is not public. This same
guide is installed at `$(brew --prefix)/share/vswitch/README.md`.
