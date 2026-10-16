# ShellWire MCP & Agent Integration

Official Model Context Protocol (MCP) client configurations, installable agent skill, and public HTTP API documentation for [ShellWire](https://shellwire.com/).

ShellWire puts useful software on the front page, with explainable rankings and discovery for people and AI assistants. Explore AI assistants, chatbots, developer tools, and everyday software by the problems they help solve, compare documented capabilities backed by dated sources, and see how each application's utility score is calculated.

> **Repository scope:** ShellWire is a hosted service (`https://shellwire.com/`) with a private application codebase. This public repository contains the official MCP Registry manifest (`server.json`), client setup guides, agent skill (`skills/shellwire/SKILL.md`), and OpenAPI specification (`openapi.json`) for connecting AI assistants and HTTP clients to ShellWire.

## Key Links

- **Website:** [https://shellwire.com/](https://shellwire.com/)
- **Assistant Connection Page:** [https://shellwire.com/connect](https://shellwire.com/connect)
- **Hosted MCP Endpoint (Streamable HTTP):** `https://shellwire.com/mcp`
- **Official MCP Registry ID:** `io.github.Kiragu-Maina/shellwire` ([Registry Record](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.Kiragu-Maina%2Fshellwire/versions/latest))
- **Leaderboard:** [https://shellwire.com/leaderboard](https://shellwire.com/leaderboard)
- **Submit an Application:** [https://shellwire.com/submit](https://shellwire.com/submit)

---

## Quick Connect (MCP)

ShellWire exposes a remote **Streamable HTTP** MCP server at `https://shellwire.com/mcp`. Public discovery, tool comparisons, and news reads are **anonymous** and require no API key or sign-in.

### Claude Code

```bash
claude mcp add --transport http shellwire https://shellwire.com/mcp
```

### Cursor / VS Code / Windsurf (`mcp.json`)

```json
{
  "mcpServers": {
    "shellwire": {
      "type": "http",
      "url": "https://shellwire.com/mcp"
    }
  }
}
```

### OpenCode (`opencode.json`)

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "shellwire": {
      "type": "remote",
      "url": "https://shellwire.com/mcp",
      "enabled": true
    }
  }
}
```

### Authenticated Builder Configuration (Optional)

If your agent is submitting an application or checking your private submission status, supply an 8-hour builder Bearer token obtained via the first-party email OTP HTTP flow (see [Builder Authentication](#builder-authentication-email-otp) below):

```json
{
  "mcpServers": {
    "shellwire": {
      "type": "http",
      "url": "https://shellwire.com/mcp",
      "headers": {
        "Authorization": "Bearer ${SHELLWIRE_BUILDER_TOKEN}"
      }
    }
  }
}
```

---

## MCP Tools Reference (8 Tools)

### Public Discovery Tools (Anonymous)

| Tool | Title | Input Arguments | Description |
| :--- | :--- | :--- | :--- |
| `search` | Search ShellWire | `{ "query": "string (max 200 chars)" }` | Search public applications, the chatbot catalog, and labeled news. Returns up to 24 stable IDs (`tool:<uuid>` or `news:<uuid>`), titles, and canonical citation URLs. |
| `fetch` | Read a public record | `{ "id": "tool:<uuid> \| news:<uuid>" }` | Fetch one public record ID returned by `search`. Includes citable text, structured metadata, and dated evidence sources. |
| `compare_tools` | Compare chatbots and tools | `{ "ids": ["tool:<uuid>", "..."] }` (1 to 4 IDs) | Compare 1 to 4 distinct public tool IDs side by side. Preserves unknown facts, source evidence, and checked dates without generated ratings. |
| `get_chatbot_brief` | Read the chatbot brief | `{ "limit": 4 }` (optional, 1 to 4) | Read the source-reviewed brief shown on the ShellWire homepage, with up to 4 labeled stories (`reporting`, `discussion`, `analysis`) and related public tools. |
| `get_submission_instructions` | Read builder submission instructions | `{}` | Return anonymous, credential-free instructions for authenticated private submissions, including exact HTTP OTP endpoints, bounded submission fields, and binary image upload rules. |

### Builder Submission Tools (Email OTP Bearer Token Required)

| Tool | Input Arguments | Description |
| :--- | :--- | :--- |
| `verify_homepage_backlink` | `{ "officialUrl": "https://example.com" }` | Verify that a visible native HTML anchor to `https://shellwire.com/` is present on the root homepage of your public HTTPS website. Cached for 5 minutes. |
| `submit_application` | `{ "name", "officialUrl", "description", "email"?, "details"?, "images"? }` | Submit an application or personal AI agent for private operator review after homepage backlink verification. Repeated pending submissions for the same normalized URL are acknowledged without duplicating records. |
| `list_my_submissions` | `{}` | Return only the authenticated builder account's private submission statuses (`pending`, `accepted`, `rejected`), optional published tool URL, and uploaded image previews. |

---

## Builder Authentication (Email OTP)

ShellWire uses first-party email OTP Bearer tokens for builder write operations. **Standard OAuth is not used**, so URL-only OAuth clients cannot use authenticated write tools without first obtaining a Bearer token over HTTPS.

1. **Request a 6-digit OTP code:**
   ```bash
   curl -X POST https://shellwire.com/api/directory/account/agent/send-code \
     -H "Content-Type: application/json" \
     -d '{"email": "you@your-domain.com"}'
   ```
2. **Exchange the code for an 8-hour Bearer token:**
   Read the 6-digit code from your authorized mailbox (or ask the human user for it) and exchange it over HTTPS:
   ```bash
   curl -X POST https://shellwire.com/api/directory/account/agent/token \
     -H "Content-Type: application/json" \
     -d '{"email": "you@your-domain.com", "code": "123456"}'
   ```
   Response:
   ```json
   {
     "access_token": "<token>",
     "token_type": "Bearer",
     "expires_in": 28800,
     "user": {
       "id": "<uuid>",
       "email": "you@your-domain.com"
     }
   }
   ```
3. **Optional binary screenshot upload (up to 4 images, max 5 MiB each):**
   Send raw binary bytes (`image/png`, `image/jpeg`, or `image/webp`; not multipart or base64):
   ```bash
   curl -X POST https://shellwire.com/api/directory/media \
     -H "Authorization: Bearer ${SHELLWIRE_BUILDER_TOKEN}" \
     -H "Content-Type: image/png" \
     --data-binary @screenshot.png
   ```
4. **Verify backlink and submit via MCP:**
   Ensure your root homepage includes a visible link to `https://shellwire.com/`, call `verify_homepage_backlink`, and then call `submit_application`.
5. **Revoke token when finished:**
   ```bash
   curl -X POST https://shellwire.com/api/directory/account/agent/revoke \
     -H "Authorization: Bearer ${SHELLWIRE_BUILDER_TOKEN}"
   ```

---

## Public REST Catalog API

In addition to MCP, ShellWire exposes public JSON endpoints under `https://shellwire.com/api/directory` (full specification in [`openapi.json`](./openapi.json)):

- `GET /api/directory/status` - Service and MCP availability status
- `GET /api/directory/tools` - Search and filter published tools (`q`, `category`, `kind`, `free`, `capability`, `limit`, `offset`)
- `GET /api/directory/tools/:slug` - Fetch a published tool record by slug with dated evidence and capabilities
- `GET /api/directory/categories` - List directory categories and tool counts
- `GET /api/directory/leaderboard` - Fetch the explainable heuristic leaderboard (`q`, `category`, `limit`) with per-factor point breakdowns
- `GET /api/directory/brief` - Fetch the homepage chatbot news brief
- `GET /api/directory/compare?ids=<uuid1>,<uuid2>` - Compare up to 4 published tools side by side
- `GET /api/directory/search?q=<query>` - Unified search across tools and labeled news
- `GET /api/directory/records/:id` - Fetch a public record (`tool:<uuid>` or `news:<uuid>`) with citable text

## Install as an Agent Skill

This repository includes a standard Agent Skill in [`skills/shellwire/SKILL.md`](./skills/shellwire/SKILL.md):

```bash
npx skills add Kiragu-Maina/shellwire-mcp
```

## License

The documentation, schemas, configuration examples, and agent skill files in this repository are released under the [MIT License](./LICENSE).
