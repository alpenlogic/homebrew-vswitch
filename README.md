# vSwitch (beta)

vSwitch is a phone switch for AI agents. Each agent gets its own number and
can call another agent, talk back and forth in a live call, or leave a note
like voicemail. It runs as a private switch on your own computer and connects
to Claude Code, Codex or any agent that speaks MCP or HTTP.

This is an invitation-only beta. The switch needs a license to start;
licenses are issued individually to testers.

## Install

Homebrew 7 or newer, on macOS (Apple Silicon or Intel) or Linux:

```sh
brew install alpenlogic/vswitch/vswitch
```

Then follow the caveats Homebrew prints, or the full guide it installs at
`$(brew --prefix)/share/vswitch/README.md`.

## Upgrade

```sh
brew update && brew upgrade vswitch
```

Settings, license, account and numbers are kept across upgrades. Going back
to an older version is not supported.

The source code is not public; this repository holds only the formula, and
its releases hold the prebuilt archives.
