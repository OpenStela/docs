# OpenStela — public API

OpenStela is a registry and market for AI characters: register the character you already made, bind its works to the record, sell or license it with the record attached.

This repository documents the **public, no-account** endpoints. Everything here is readable by anyone; nothing here writes.

Base URL: `https://app.openstela.io`

## What the record proves — and what it does not

A registry record proves that a specification (and each bound work) **existed in this form at a given time**, and that its fingerprint was anchored on a public blockchain. It does **not** prove who created the character, that nothing in it infringes, or that the holder may sell it. Treat every endpoint below as evidence, not title.

## Endpoints

### `GET /api/verify/{sha256}`

Look up a fingerprint (64 hex chars). Works for specification versions, bound works, evidence records, channel verifications, contract texts and transfer events.

```
curl https://app.openstela.io/api/verify/708613fefe5bcb31a5bdc8cf4ff139b160b88dcc4072429a44e2ad1d547531c8
```

Response (abridged; a small JPEG thumbnail of the character is also returned as a data URI when the leaf is a specification version):

```json
{
  "found": true,
  "type": "version",
  "id": "AVL-MWKRYG",
  "leaf": "708613fe…7531c8",
  "registered_at": "2026-08-09T16:24:20+00:00",
  "pack_version": "1.1.0",
  "anchored": true,
  "chain": { "ok": true, "proof_ok": true, "root_on_chain": true, "root": "…", "tx": "0x…", "chain": "…", "explorer": "…" },
  "disclaimer": {
    "proves": "this specification has existed on this platform, unaltered, since 2026-08-09",
    "does_not_prove": ["that the holder is the author", "…"]
  }
}
```

`type` is one of `version`, `work`, `evidence`, `channel`, `contract`, `transfer`. `chain.proof_ok` means the Merkle proof was recomputed and matched; `root_on_chain` means the root was read back from the chain, not from our database. Unknown hashes return `{"found": false}`. The endpoint never returns specification text or prompts.

### `GET /api/public/character/{AVL-XXXXXX}`

The public page data for a listed character: card (name, type, holder handle), reference image URLs, bound works with their public evidence summary, verified channels, registry events (versions, transfers, licences).

- Listed characters return the full public payload.
- Registered but unlisted characters return `{"unlisted": true, "card": …, "identity": …, "events": […]}` — the record stays resolvable after a transfer even if the new holder has not relisted.
- Specification text, prompts and palette values are never included.

### `GET /api/public/user/{handle}`

A creator's public profile: handle, display name, bio, and the characters they have listed publicly. Email and internal IDs are never included.

### `GET /badge/{AVL-XXXXXX}.svg`

A 22px-high SVG badge. Listed characters get `OpenStela | registered` or `OpenStela | anchored YYYY-MM-DD`; anything else gets a grey `no public record` badge (unlisted and nonexistent IDs are indistinguishable by design). Cached for one hour.

Markdown:

```markdown
[![OpenStela](https://app.openstela.io/badge/AVL-XXXXXX.svg)](https://app.openstela.io/c/AVL-XXXXXX)
```

HTML:

```html
<a href="https://app.openstela.io/c/AVL-XXXXXX"><img src="https://app.openstela.io/badge/AVL-XXXXXX.svg" alt="OpenStela: registered character AVL-XXXXXX" height="22"></a>
```

The badge deliberately never says "verified owner" or "certified". If you see one that does, it is not ours.

### `GET /api/market/list`

All publicly listed characters (cards only).

## Public pages

| URL | What it is |
|---|---|
| `/c/{AVL-XXXXXX}` | Character page |
| `/u/{handle}` | Creator page |
| `/verify?h={sha256}` | Human-readable fingerprint check |
| `/market` | The market |

## MCP server (for AI assistants)

Holders can expose their own character packs to an AI assistant through the OpenStela MCP server (read-only: list characters, fetch the per-platform anchor prompt). It requires a personal API key created under Account → API keys in the app, and runs locally:

```json
{
  "mcpServers": {
    "openstela": {
      "command": "python",
      "args": ["/path/to/mcp_server.py"],
      "env": { "AVATAR_LAB_URL": "https://app.openstela.io", "AVATAR_LAB_TOKEN": "alk_…" }
    }
  }
}
```

The server source ships with the app; ask at support@openstela.io for the current file until it is published here.

## Rate limits and etiquette

No authentication, no keys, no published rate limit. Please cache badge and verify responses (both are safe to cache for an hour) and identify your client with a User-Agent.

## Contact

support@openstela.io · https://openstela.io
