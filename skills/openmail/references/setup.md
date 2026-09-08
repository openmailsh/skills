# OpenMail setup

**Using OpenClaw?** Skip this skill. Install the plugin: `openclaw plugins install clawhub:@openmail/openclaw` — see https://docs.openmail.sh/integrations/openclaw.

## Install

```bash
npm install -g @openmail/cli   # Node.js 20+; or run: npx @openmail/cli <command>
```

## Authenticate and create the default inbox

Get an API key at https://console.openmail.sh (free, no card). Pass it once:

```bash
openmail init --api-key om_... --mailbox-name "agent" --display-name "Agent"
```

This creates the inbox and saves both the key and the inbox (id, address) as defaults in `~/.openmail-cli/state.json`. Omit `--mailbox-name`/`--display-name` to be prompted. After this, every command works without `--api-key`, and `send`, `threads list`, and `messages list` need no `--inbox-id`.

Override the saved key any time: `--api-key`, then `OPENMAIL_API_KEY`, then `OPENMAIL_API_KEY=...` in `./.env` take precedence over the state file.

Verify: `openmail inbox list`

## Hermes

Hermes strips environment variables from the shells it runs, so the key has to be declared, not just exported. This skill declares `OPENMAIL_API_KEY`; Hermes asks for it the first time the skill loads and stores it in `~/.hermes/.env`. To set it yourself, add a line to that file:

```bash
echo 'OPENMAIL_API_KEY=om_...' >> ~/.hermes/.env
```

Then `openmail init --mailbox-name "agent" --display-name "Agent"` (no `--api-key`; the CLI reads the env var). Do not put an account-wide key here; use a pod-scoped key so the agent, and anything it spawns, stays inside one pod.

## More inboxes

```bash
openmail inbox create --mailbox-name "support" --display-name "Support"
```

Target one inbox with `--inbox-id inb_...` on `send`, `threads list`, and `messages list`, or set `OPENMAIL_INBOX_ID`.

## Reset

Delete `~/.openmail-cli/state.json` to forget both the saved key and the default inbox. Also unset `OPENMAIL_API_KEY` (or remove it from `./.env`) if you set it.
