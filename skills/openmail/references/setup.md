# OpenMail Setup

## One command

```bash
npx @openmail/cli setup --agent claude-code
```

This handles everything:
- Opens your browser to sign in (or prompts for your API key)
- Asks for a mailbox name and display name
- Creates your inbox
- Writes credentials to `~/.claude/openmail.env`
- Writes the skill file to `~/.claude/skills/openmail/SKILL.md`

No global install required — `npx` fetches the CLI automatically.

## Requirements

- Node.js 20+
- An OpenMail account (free at [console.openmail.sh](https://console.openmail.sh), no credit card)

## Verify

```bash
npx @openmail/cli status
```

## Multiple inboxes

To create additional inboxes after setup:

```bash
npx @openmail/cli inbox create --mailbox-name "support" --display-name "Support"
```

## Remove

```bash
npx @openmail/cli setup --agent claude-code --reset
```
