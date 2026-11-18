# OpenMail CLI & API Reference

Base URL: `https://api.openmail.sh`  
Authentication: `Authorization: Bearer <OPENMAIL_API_KEY>`  
All responses are JSON.

---

## CLI commands

### Send email

```bash
openmail send \
  --to "recipient@example.com" \
  --subject "Subject" \
  --body "Plain text body."

# Reply in thread (subject ignored)
openmail send \
  --to "recipient@example.com" \
  --thread-id "thr_..." \
  --body "Reply body."

# With HTML and attachments
openmail send \
  --to "recipient@example.com" \
  --subject "Report" \
  --body "See attached." \
  --body-html "<p>See attached.</p>" \
  --attach ./report.pdf \
  --attach ./data.csv
```

Flags:
- `--to` (required) — recipient address
- `--subject` (required unless `--thread-id` is set) — email subject
- `--body` (required) — plain text body
- `--body-html` — optional HTML body
- `--thread-id` — reply in an existing thread
- `--attach` — file path to attach (repeatable)
- `--inbox-id` — override default inbox

Response:

```json
{
  "id": "msg_...",
  "threadId": "thr_...",
  "inboxId": "inb_...",
  "direction": "outbound",
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

### Email drafts

```bash
openmail drafts create --inbox-id "<inbox-id>" --to "person@example.com" --subject "Proposal" --body "Please review." --attach "./proposal.pdf" --json
openmail drafts create --inbox-id "<inbox-id>" --thread-id "<thread-id>" --body "Thanks for the update." --json
openmail drafts list --pod-id "<pod-id>" --inbox-id "<inbox-id>" --query Proposal --limit 20 --offset 0 --json
openmail drafts get --draft-id "<id>" --json
openmail drafts send --draft-id "<id>" --updated-at "<reviewed updatedAt>" --json
```

The inbox is required at creation. All content fields are optional. Use repeatable `--cc`, `--bcc`, and `--attach` flags, and `--body-html` for HTML. For a local inline image, put `cid:inline-<lowercase-UUID>@openmail` in the HTML and pass `--inline-image 'inline-<lowercase-UUID>@openmail=./photo.png'` with the same ID. Use `--thread-id` for a reply in the sending inbox, or `--source-thread-id` alone for a draft that will start a new conversation. The latter takes its recipient, subject, and content from the draft you supply. Both appear in their source thread.

Listing defaults to all accessible inboxes. Combine `--inbox-id`, `--pod-id`, `--source-thread-id`, and `--query`. Get the draft by ID and review its owning inbox, recipients, full content, attachments and any thread association before sending. An optional `--inbox-id` on send must match the owner.

### Threads

```bash
# List unread threads (preferred for checking new mail)
openmail threads list --is-read false

# List all threads
openmail threads list

# List with pagination
openmail threads list --limit 20 --offset 0

# Get messages in a thread (oldest-first)
openmail threads get --thread-id "thr_..."

# Mark thread as read
openmail threads read --thread-id "thr_..."

# Mark thread as unread
openmail threads unread --thread-id "thr_..."
```

Thread object:

```json
{
  "id": "thr_...",
  "inboxId": "inb_...",
  "subject": "Re: Hello",
  "lastMessageAt": "2024-01-01T00:00:00.000Z",
  "isRead": false,
  "messageCount": 3
}
```

### Messages

```bash
# List inbound messages
openmail messages list --direction inbound --limit 20

# List outbound messages
openmail messages list --direction outbound

# With pagination
openmail messages list --direction inbound --limit 50 --offset 100
```

Message object:

```json
{
  "id": "msg_...",
  "threadId": "thr_...",
  "inboxId": "inb_...",
  "fromAddr": "Alice <alice@example.com>",
  "toAddr": "agent@openmail.sh",
  "subject": "Hello",
  "bodyText": "Plain text content",
  "bodyHtml": "<p>HTML content</p>",
  "direction": "inbound",
  "attachments": [
    {
      "filename": "report.pdf",
      "url": "https://...",
      "sizeBytes": 12345
    }
  ],
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

### Inboxes

```bash
# List all inboxes
openmail inbox list

# Create an inbox
openmail inbox create --mailbox-name "support" --display-name "Support"

# Get inbox details
openmail inbox get --inbox-id "inb_..."

# Delete an inbox
openmail inbox delete --inbox-id "inb_..."
```

Inbox object:

```json
{
  "id": "inb_...",
  "address": "support@example.openmail.sh",
  "displayName": "Support",
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

---

## REST API

All endpoints require `Authorization: Bearer <OPENMAIL_API_KEY>`.

### POST /v1/inboxes/{id}/send

Send an email. `Idempotency-Key` header (UUID) is optional; pass one to make retries safe.

```http
POST /v1/inboxes/{id}/send
Authorization: Bearer om_...
Content-Type: application/json
Idempotency-Key: <uuid>

{
  "to": "recipient@example.com",
  "cc": ["colleague@example.com"],
  "bcc": ["crm@hubspot.example"],
  "subject": "Hello",
  "body": "Plain text body.",
  "bodyHtml": "<p>HTML body.</p>",
  "threadId": "thr_...",
  "attachments": [
    {
      "filename": "file.pdf",
      "contentType": "application/pdf",
      "data": "<base64>"
    }
  ]
}
```

`cc` and `bcc` are optional arrays. `bcc` recipients get the message but are hidden from the To and Cc recipients; use it for CRM logging addresses. Every `to`, `cc`, `bcc`, and `replyTo` address is checked against the inbox's outbound sender rules — one denied address rejects the whole send with `403 recipient_not_permitted`.

### Saved drafts

| Method and path | Request and behavior |
| --- | --- |
| `POST /v1/drafts` | Requires `inboxId`. Accepts `to`, `cc`, `bcc`, `subject`, `body`, `bodyHtml`, `threadId`, and `sourceThreadId`. Returns the saved draft. |
| `GET /v1/drafts` | Returns `{data, total}` for accessible inboxes. Combine `inboxId`, `podId`, `sourceThreadId` and text search `q` (subject, recipient or body). Pagination accepts `limit` (1–100, default 50) and `offset` (default 0). |
| `GET /v1/drafts/{draftId}` | Returns the owning inbox, saved content, thread associations, `updatedAt`, and attachment metadata including filename, MIME type, byte size, and a download URL when available. Inline images also carry `contentId`. |
| `POST /v1/drafts/{draftId}/send` | Requires the exact reviewed `updatedAt`. Optional `inboxId` must match the owner. Sends the saved content and returns the sent message and thread IDs. |

Create accepts JSON for text or HTML and multipart form data for uploads. Repeat the `attachments` field for files or `inlineImages` for PNG, JPEG, GIF or WebP images. Give each inline image the multipart filename `inline-<lowercase-UUID>@openmail` and reference that ID with `cid:` in the HTML. Attachments and images share a 25 MiB limit. Supported embedded or public remote images in HTML are imported when saving the draft.

A reply uses `threadId` and inherits its subject from that thread. A forward can use `sourceThreadId` alone to appear in the source thread and send into a new one. A referenced thread must belong to the sending inbox. When both IDs are present, they must match.

A successful send removes the draft. On `409 draft_changed`, get the current version and review it again. On `409 draft_sending`, check the inbox and draft before considering another send. On `502 send_failed`, the draft is retained and its version may advance. Review it again before retrying. A different sending inbox returns `400 draft_inbox_locked`.

### GET /v1/inboxes/{id}/threads

```
GET /v1/inboxes/{id}/threads?isRead=false&limit=20&offset=0
```

### PATCH /v1/threads/{id}

```http
PATCH /v1/threads/{id}
Content-Type: application/json

{ "isRead": true }
```

### GET /v1/threads/{id}/messages

Returns messages in a thread, sorted oldest-first.

### GET /v1/inboxes/{id}/messages

```
GET /v1/inboxes/{id}/messages?direction=inbound&limit=20&offset=0
```

### GET /v1/attachments/{messageId}/{filename}

Returns attachment data. Signed URL included in message payload — fetch promptly, URLs expire.

---

## Rate limits

| Resource | Limit |
|---|---|
| Inbox creation | 100 per day |
| Send per inbox | 10 per minute, 200 per day |
| API requests | 1,000 per minute |

---

## Idempotency

The send endpoint accepts an optional `Idempotency-Key` header. Without it the API generates one and sends once. If your code retries on timeouts, pass a unique UUID per send attempt: retrying with the same key within 24 hours returns the original response without resending.

The CLI generates idempotency keys automatically. For direct API calls, generate a UUID:

```bash
python3 -c "import uuid; print(uuid.uuid4())"
```
