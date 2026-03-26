# OpenMail Agent Skills

Agent skills for [OpenMail](https://openmail.sh) — give AI agents a real email address for sending and receiving email.

License: MIT

## Install

```bash
npx skills add openmailsh/skills
```

Or install a specific skill:

```bash
npx skills add openmailsh/skills@openmail
```

---

## Available Skills

| Skill | Description |
|---|---|
| [openmail](./skills/openmail/SKILL.md) | Dedicated email address for sending, receiving, and replying in threads. Use when the agent needs to send email to external services, receive replies, sign up for services, handle support tickets, or interact with any human institution via email. |

---

## CLI Setup

Requires the OpenMail CLI (`openmail`):

```bash
npm install -g @openmail/cli
```

Then run setup to create your inbox and configure credentials:

```bash
openmail setup
```

Get your API key at [console.openmail.sh](https://console.openmail.sh) (free, no credit card required).

---

## Documentation

- [Quickstart](https://docs.openmail.sh/quickstart)
- [CLI reference](https://docs.openmail.sh/quickstart)
- [API reference](https://docs.openmail.sh/api-reference/introduction)
- [Concepts](https://docs.openmail.sh/concepts/inboxes)

**Links:** [openmail.sh](https://openmail.sh) · [console.openmail.sh](https://console.openmail.sh) · [@openmail/cli on npm](https://www.npmjs.com/package/@openmail/cli) · [Discord](https://discord.gg/eFfQFMZbsK)
