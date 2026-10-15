# spotify-mcp (Roybox/Tools)

Third-party clone of https://github.com/marcelmarais/spotify-mcp-server — **not Roy's repo, never push to its remote.**
Registered as a user-scope stdio MCP server: `node ~/Downloads/Roybox/Tools/spotify-mcp/build/index.js`.
Credentials live in `spotify-config.json` (gitignored, mode 600). Rebuild after any pull: `npm install && npm run build`.

## Lesson log

### 2026-09-06 — Spotify's official remote MCP cannot be used from Claude Code
- **Symptom:** `claude mcp add --transport http spotify https://mcp-gateway-external-pilot.spotify.net/mcp`
  adds fine, then `/mcp` → Authenticate fails with
  `Incompatible auth server: does not support dynamic client registration`.
- **Root cause:** two independent blockers.
  1. `accounts.spotify.com` does not implement RFC 7591 dynamic client registration, so Claude Code
     cannot self-register an OAuth client. `claude mcp add --client-id ...` works around this one.
  2. Claude Code's MCP OAuth callback listener builds its redirect as
     `http://localhost:<port>/callback` (confirmed in the 2.1.263 binary). Spotify's dashboard
     **rejects `localhost`** and requires an explicit loopback IP (`127.0.0.1` / `::1`). Nothing on
     the Claude Code side lets you change the host, only `--callback-port` changes the port.
     Blocker 2 has no workaround.
- **Fix:** use the local stdio server instead. It runs its own OAuth (`npm run auth`) against
  `http://127.0.0.1:8888/callback`, bypassing Claude Code's OAuth entirely.
- **Worth knowing:** the remote endpoint's `/.well-known/oauth-protected-resource` does advertise
  `playlist-modify-private` (but *not* `playlist-modify-public`), so it is a write-capable server —
  it is only the auth handshake that is unusable. Re-test if Claude Code ever switches its MCP
  redirect host to `127.0.0.1`.
- **Also:** the upstream README says Spotify Premium is required. That is true only for the playback
  tools; playlist create/add works on a free account.
