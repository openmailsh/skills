# OpenMail setup

**Using OpenClaw?** Skip this skill. Install the plugin: `openclaw plugins install clawhub:@openmail/openclaw` — see https://docs.openmail.sh/integrations/openclaw.

## Install

```bash
npm install -g @openmail/cli   # Node.js 20+; or run: npx @openmail/cli <command>
```

## Authenticate

Get an API key at https://console.openmail.sh (free, no card). The CLI reads it from `--api-key`, then `OPENMAIL_API_KEY`, then `OPENMAIL_API_KEY=...` in `./.env`. Persist it:

```bash
export OPENMAIL_API_KEY=om_...   # add to your shell profile, or put the line in ./.env
```

## Create the default inbox

```bash
openmail init --mailbox-name "agent" --display-name "Agent"
```

Uses the key above to create the inbox and saves its id and address as the default in `~/.openmail-cli/state.json`. Omit the flags to be prompted. After this, `send`, `threads list`, and `messages list` need no `--inbox-id`.

Verify: `openmail inbox list`

## More inboxes

```bash
openmail inbox create --mailbox-name "support" --display-name "Support"
```

Target one inbox with `--inbox-id inb_...` on `send`, `threads list`, and `messages list`, or set `OPENMAIL_INBOX_ID`.

## Reset

Delete `~/.openmail-cli/state.json` to forget the default inbox. Unset `OPENMAIL_API_KEY` (or remove it from `./.env`) to sign out.
