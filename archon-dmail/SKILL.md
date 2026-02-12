---
name: archon-dmail
description: Send and receive encrypted messages between DIDs. End-to-end encrypted, decentralized messaging with attachments. Requires existing Archon identity (see archon-id skill).
---

# Archon Dmail

Send and receive encrypted messages (dmail) between DIDs. Messages are end-to-end encrypted, stored on the DID registry, and include support for attachments.

## Prerequisites

- Archon identity set up (run `archon-id` skill first)
- Environment configured (`~/.archon.env` with `ARCHON_PASSPHRASE` and optionally `ARCHON_WALLET_PATH`)

## Quick Start

```bash
# Source environment
source ~/.archon.env
export ARCHON_WALLET_PATH="${ARCHON_WALLET_PATH:-$HOME/clawd/wallet.json}"

# Send a message
./scripts/send.sh "did:cid:recipient..." "Subject line" "Message body"

# Check for new messages
./scripts/refresh.sh

# List inbox
./scripts/list.sh

# Read a message
./scripts/read.sh "did:cid:message..."
```

## Scripts

### send.sh - Send a Message

```bash
./scripts/send.sh <recipient-did> <subject> <body> [cc-did...]
```

Creates and sends an encrypted message. Returns the notice DID.

**Examples:**
```bash
# Simple message
./scripts/send.sh "did:cid:alice..." "Hello" "How are you?"

# With CC
./scripts/send.sh "did:cid:alice..." "Meeting" "Let's sync up" "did:cid:bob..."
```

### list.sh - List Messages

```bash
./scripts/list.sh [filter]
```

Lists messages in your inbox. Optional filter: `unread`, `sent`, or any tag.

**Output:** Subject, sender, date, and tags for each message.

### read.sh - Read a Message

```bash
./scripts/read.sh <dmail-did>
```

Displays full message content including attachments list.

### refresh.sh - Check for New Messages

```bash
./scripts/refresh.sh
```

Polls the network for new dmails and imports them to inbox.

### reply.sh - Reply to a Message

```bash
./scripts/reply.sh <dmail-did> <body>
```

Replies to a message (uses original subject with "Re:" prefix, sets reference).

### forward.sh - Forward a Message

```bash
./scripts/forward.sh <dmail-did> <recipient-did> [body]
```

Forwards a message to another DID with optional additional text.

### archive.sh - Archive a Message

```bash
./scripts/archive.sh <dmail-did>
```

Removes `inbox` and `unread` tags, adds `archive` tag.

### delete.sh - Delete a Message

```bash
./scripts/delete.sh <dmail-did>
```

Removes message from inbox (does not revoke the DID).

### attach.sh - Add Attachment

```bash
./scripts/attach.sh <dmail-did> <file-path>
```

Adds a file attachment to an existing dmail (before sending).

### get-attachment.sh - Download Attachment

```bash
./scripts/get-attachment.sh <dmail-did> <attachment-name> <output-path>
```

Downloads an attachment from a dmail.

## Message Format

Dmails are JSON files with this structure:

```json
{
    "to": ["did:cid:recipient1", "did:cid:recipient2"],
    "cc": ["did:cid:cc-recipient"],
    "subject": "Subject line",
    "body": "Message body text",
    "reference": "did:cid:original-message"
}
```

- **to**: Array of recipient DIDs (required, at least one)
- **cc**: Array of CC'd DIDs (optional)
- **subject**: Subject line (required)
- **body**: Message body (required)
- **reference**: DID of message being replied to (optional)

## Tags

Dmails use tags for organization:

- `inbox` - In inbox (default for received)
- `unread` - Not yet read
- `sent` - Messages you sent
- `archive` - Archived messages
- Custom tags via `file-dmail` command

## Advanced Usage

### Direct Keymaster Commands

```bash
# Create dmail from JSON file
npx @didcid/keymaster create-dmail message.json

# Send existing dmail
npx @didcid/keymaster send-dmail <dmail-did>

# Tag a message
npx @didcid/keymaster file-dmail <dmail-did> "inbox,important"

# Import external dmail
npx @didcid/keymaster import-dmail <dmail-did>
```

### Ephemeral Messages

On hyperswarm registry, dmails can be set to expire:

```bash
npx @didcid/keymaster create-dmail message.json -r hyperswarm
# Then set validUntil via update-dmail
```

## Troubleshooting

**"Wallet not found"**: Ensure `ARCHON_WALLET_PATH` points to your wallet.json

**"No passphrase"**: Set `ARCHON_PASSPHRASE` in environment or ~/.archon.env

**Messages not arriving**: Run `refresh.sh` to poll for new dmails

**Recipient can't decrypt**: Ensure you're using their correct DID (not an alias)
