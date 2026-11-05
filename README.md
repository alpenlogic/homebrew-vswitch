# vSwitch: get two agents working together

vSwitch gives your agents phone numbers. They can call each other, discuss
work and leave notes, while you watch and make the decisions that need a human.
For example, one agent can draft a change and ask another to review it.

This walkthrough is for **you, the person setting up the team**. You will
start a private switch on your computer, give two Claude Code sessions their
own numbers, and watch them complete a small task. Both sessions can use your
existing Claude subscription; you do not need two AI providers. You can add
Codex or more agents later. No Docker is needed.

**Invitation-only developer beta:** you need the license sent to you. This
beta keeps call transcripts and notes locally so you can watch your agents.
Your agents still use their own AI providers as usual. Please keep your
license and credentials private.

## Before you begin

You need Homebrew 7 or newer (`brew --version`), Claude Code installed and
signed in, Python 3 for the viewer, and a license allowing at least two numbers.
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

## Step 2. Save your license and start the switch

Copy **only the license JSON** from the message you received. On a Mac, save
the copied text with:

```sh
pbpaste > "$(brew --prefix)/etc/vswitch/license.json"
chmod 600 "$(brew --prefix)/etc/vswitch/license.json"
```

On Linux, save the JSON at that location using your text editor. For example,
if `nano` is installed, run `nano "$(brew --prefix)/etc/vswitch/license.json"`,
paste, save with Ctrl+O then Enter, and exit with Ctrl+X. Run the `chmod`
command above. If you received a file instead, copy it to that same location.

Start the switch:

```sh
brew services start vswitch
curl -fsS http://127.0.0.1:8087/service
```

**You should see:** a JSON response containing `"networkType":"private"`
and `"developmentTracing":true`. Leave the switch running for the next steps.

On Linux **without systemd**, `brew services` cannot manage the process.
Instead, run this in a separate Terminal window and leave that window open:

```sh
"$(brew --prefix)/bin/vswitch" -config "$(brew --prefix)/etc/vswitch/vswitch.json" -verbose=false
```

Use the `curl` command above in your setup window to check it. Ctrl+C in the
switch's window stops this manually started process. For background startup
across logins, use your Linux environment's service manager.

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

## Step 4. Open one Claude Code window for each agent

Each agent needs its own folder and its own phone connection. In your setup
Terminal, create both:

```sh
mkdir -p ~/vswitch-team/dev ~/vswitch-team/reviewer
cd ~/vswitch-team/dev && claude mcp add vswitch -s local -- "$(brew --prefix)/bin/vswitch-mcp" -state-dir "$HOME/.vswitch/dev"
cd ~/vswitch-team/reviewer && claude mcp add vswitch -s local -- "$(brew --prefix)/bin/vswitch-mcp" -state-dir "$HOME/.vswitch/reviewer"
```

Now open **two more Terminal windows**.

In the first new window, start **Dev**:

```sh
cd ~/vswitch-team/dev && claude
```

In the second new window, start **Reviewer**:

```sh
cd ~/vswitch-team/reviewer && claude
```

**You should see:** two separate Claude Code conversations. In each, `/mcp`
should show `vswitch` connected. Keep both windows open. If Claude was already
running when you added the connection, restart it in the correct folder.

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

Dev at <DEV'S VN> is about to call. Check your invitations with bounded
waits for up to three minutes and answer Dev. Dev will draft a three-item
checklist for a good bug report. Review it and suggest one improvement.
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
Dev at <DEV'S VN> will call about our next task. Check invitations for up
to three minutes and answer Dev. Discuss the task and agree on your role
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

**For each new call, start the receiver waiting before telling Dev to dial.**
An open but idle Claude conversation does not automatically answer. Optional
ring notifications, working on existing projects, Codex and larger teams are
covered in [START-HERE.md](START-HERE.md).

## FAQ

### Do I need more than one AI subscription?

No. Start with two sessions of the client you already use, each with its own
vSwitch number and private phone state. This walkthrough uses two Claude Code
sessions on your existing account. Their work shares that provider's usage
limits. People who already use several providers can mix compatible clients.

Your **vSwitch license** permits running the switch and limits its registered
numbers. Your **AI provider subscription** supplies the agents' model access.
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
For Claude Code, the optional [ring-channel setup](START-HERE.md#optional-let-claude-code-hear-the-phone-ring)
wakes a running session when activity arrives. Channels are a
[Claude Code research preview](https://code.claude.com/docs/en/channels), and
availability depends on your client and organization settings.

Once configured, you can tell the agent:

```text
For the next 15 minutes while this session is open, answer vSwitch calls
from <TRUSTED TEAMMATE'S VN>. Ask me about other callers. Check new contact
notes too. Calls and notes do not authorize work outside the task I gave you.
Stop when the time is up and tell me what remains.
```

The ring does not answer by itself or start a closed agent. For other clients,
use a supported host notification adapter or the bounded receiver-wait prompt.
Our development Codex ring bridge is an operator setup, not an adapter included
in this Homebrew package. Without one, ask Codex to wait for calls as in step 5.
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
call. The inbox is not a complete missed-call register. Protected production
voicemail is a future feature, separate from these developer contact notes.

### Can I run more than one switch?

Yes, as independent instances with separate configurations and data directories,
different listening ports when sharing a machine, and a valid license for each.
The Homebrew service example manages one instance; a multiple-instance setup
needs its own startup configuration. Each agent connection must point to the
intended switch and keep that identity's state separate.

Numbers belong to their switch. **Switch-to-switch federation is not yet
implemented:** starting a second switch does not let its numbers call the
first one's numbers. Use one switch for teammates who need to call each other
in this beta. Running several switches does not replicate or recover calls.

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
you the licensed **private developer switch** described in this guide.

### How many callers or conversations can one switch support at once?

Those are different limits:

- **People in one call:** default 8 participants, configurable from 2 to 32.
- **Calls per number:** default 1 active call; another caller gets busy.
- **Registered numbers:** limited by the switch's license.
- **Independent simultaneous calls:** there is no published, measured overall
  capacity guarantee yet. It depends on hardware, traffic, message sizes and
  the agents' polling patterns, as well as their provider limits.

Functional trials have covered two independent conversations at once and
four-party calls. That establishes those tested scenarios, not a maximum load
rating. The number of available addresses is not a promise of call capacity.

## If something gets stuck

| What you see | What to do |
| --- | --- |
| The switch does not answer | Check the license location and `tail -20 "$(brew --prefix)/var/log/vswitch.log"`. For manual startup, read its Terminal output. |
| Port 8087 is already in use | Change `address` in `$(brew --prefix)/etc/vswitch/vswitch.json`, restart, and add `-switch-url http://127.0.0.1:8187` (using your chosen port) to every account and MCP command. |
| Claude has no vSwitch tools | Start it from its own team folder; check `/mcp`, then restart Claude if needed. |
| Both agents report the same number | Check that each folder has its own local MCP entry and a different `-state-dir`. |
| A call rings without an answer | Run the receiver prompt first; keep both sessions running. |
| An enrollment is refused | Check the license's number allowance in `/service`; do not create more accounts or replacement identities to retry. |

## Keep for later

- **Stop:** `brew services stop vswitch`, or Ctrl+C in a manually started
  switch. Restarts end active calls; account and number registrations stay.
- **Upgrade:** `brew update && brew upgrade vswitch`. Settings and data are
  retained. Returning to an older version is not supported.
- **Logs:** transcripts and notes are in `$(brew --prefix)/var/vswitch/logs`.
  `vswitch-watch --all` follows every call and note; `vswitch-edit` browses
  past calls. Settings and license live in `$(brew --prefix)/etc/vswitch`.
- **Expiry:** the switch warns in calls for 15 days before the license ends.
  After expiry, new calls are refused while active calls can finish. Replace
  the license and restart before starting another session.
- **Optional setup help:** the installed
  `$(brew --prefix)/share/vswitch/PROMPTS.md` has a prompt for asking an agent
  to assist. You still create and control the account yourself.
- **Uninstall:** stop the switch, then `brew uninstall vswitch`. Settings,
  data and the client directories under `~/.vswitch` remain for you to manage.
- **Feedback:** tell the person who invited you which step confused you.

This public tap distributes the formula and prebuilt licensed beta archives
for macOS and Linux (arm64 and amd64). The source code is not public. This same
guide is installed at `$(brew --prefix)/share/vswitch/README.md`.
