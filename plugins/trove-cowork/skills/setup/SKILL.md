---
name: trove-setup
description: First-run setup check for the Trove plugin. Verifies that the bundled Trove MCP connector is authorized by calling the whoami tool and reporting the signed-in account and plan tier. Run this right after the plugin is installed, or whenever Trove tools return 401 Unauthorized or the user says Trove "isn't connected".
---

# Trove — Setup

Trove ships its own MCP connector (`trove` → `https://heytrove.ai/api/v1/mcp`). Installing this plugin adds the skill, the four sub-agents, and that connector in one step — but the connector still needs the user's **OAuth authorization** before any tool call will work. Claude prompts them to sign in during install; this file is how you confirm that the sign-in actually landed.

Everything here is an MCP tool call. Cowork has no shell, no filesystem, and no `trove` CLI — never suggest one.

## Step 1 — Verify the connection

Call the `whoami` tool on the `trove` MCP server. Nothing else. Do not call `library_summary`, do not spawn sub-agents — this is a connectivity check, not a research run.

## Step 2 — Report the result

**On success**, `whoami` returns the signed-in account and plan. Report it in one or two lines:

```
✓ Trove is connected — signed in as <email> (<FREE | PRO>).
```

Then, in one sentence, tell them how to use it: just ask a question about their library ("what do my books say about X?") and the Trove skill takes over. Stop there — no tour, no feature list.

**On failure**, match the error:

| What you see | What it means | Tell the user |
|---|---|---|
| `401 Unauthorized`, "not authenticated", "invalid token" | Never authorized, or the OAuth grant was revoked / expired | Open **Customize → Connectors**, find **Trove**, and click **Connect** / **Reconnect**. A browser tab opens — sign in with the same account used at `heytrove.ai`, approve the `library:*` and `marketplace:*` scopes, then come back and say "check again". |
| Tool not found / no `trove` server listed | The bundled connector didn't install | Reinstall the Trove plugin from **Customize → Plugins**, then re-run this check. |
| Timeout or 5xx | Transient server issue | Say it's a temporary Trove outage and retry once. Do **not** call it an auth problem or a plan limit. |

Never guess the account or tier — those values come from `whoami` or they don't get reported at all. If the user has no Trove account yet, point them at `https://heytrove.ai` to sign up first; the connector can only authorize an existing account.

## Step 3 — Help

If reconnecting doesn't fix it:

- Support: https://heytrove.ai/support
- Email: support@heytrove.ai
- Revoke or inspect the authorization at https://heytrove.ai/settings

Setup is a one-time thing. Once `whoami` succeeds, don't run this check again unless a Trove tool comes back `401`.
