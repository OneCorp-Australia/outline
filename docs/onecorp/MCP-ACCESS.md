# OneCorp KB — MCP access

This fork ships a built-in **Model Context Protocol (MCP) server** so AI agents
(Claude Code, Claude Desktop, etc.) can read and write the OneCorp knowledge base
at https://kb.onecorp.au with each user's own permissions. This is a OneCorp
addition on top of upstream Outline — see also [CUSTOM_BUILD.md](CUSTOM_BUILD.md).

## Endpoint

- **MCP URL:** `https://kb.onecorp.au/mcp` (streamable HTTP transport)
- **Auth:** per-user OAuth 2.1 — PKCE (S256) + refresh token, with Dynamic Client
  Registration. No shared API key; every call runs as the signed-in user, so MCP
  access inherits that user's collection/document permissions.
- **Discovery:** `/.well-known/oauth-authorization-server/mcp` and
  `/.well-known/oauth-protected-resource/mcp` (served from `server/routes/index.ts`).

## Workspace gate

MCP is gated by the `TeamPreference.MCP` workspace preference (defaults to **on**;
migration `server/migrations/20260320000000-add-team-guidance-mcp.js`). An admin can
toggle it in workspace settings. If it's off, the `/mcp` and `.well-known` routes
return 404.

## Connect from Claude Code

```bash
claude mcp add --transport http onecorp-kb https://kb.onecorp.au/mcp
```

Then run `/mcp` in Claude Code and complete the browser OAuth flow. The connection
appears as `onecorp-kb`. (Claude Desktop / other clients: add the same URL as a
streamable-HTTP MCP server and authenticate in-browser.)

## Tools exposed (~17)

- **Collections:** list, create, update, delete; list a collection's documents
- **Documents:** create, update, delete, fetch, search, move, list
- **Comments:** create, update, delete, list
- **Attachments:** create
- **Users:** list

## Where the code lives

- `server/routes/mcp/` — MCP server + tool handlers
- `server/routes/oauth/` — OAuth authorize/token/callback
- `server/routes/api/oauthClients`, `server/routes/api/oauthAuthentications` — DCR + grants
- `server/routes/index.ts` — `.well-known` discovery + `/mcp` mount (gated on `TeamPreference.MCP`)

## Notes

- Keep the OAuth + MCP routes when merging upstream — they are not in
  `outline/outline`. If upstream ever ships a native MCP server, reconcile against it.
- Agent guidance: document markdown must not start with an H1 (the title is a
  separate field); start the body with text or a lower-level heading.
