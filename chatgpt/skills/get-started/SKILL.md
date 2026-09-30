---
name: get-started
description: First-run check for the Trove app. Confirms the Trove connection works by calling whoami and reports the signed-in account and plan. Use right after the user connects Trove, or when a Trove tool returns 401 Unauthorized.
---

# Trove — Get started

CandleKeep is the former name of Trove.

## Step 1 — Verify the connection

Call the `whoami` tool on the Trove server. Nothing else — this is a connection check, not a research run.

## Step 2 — Report the result

**On success**, report it in one line:

```
Trove is connected — signed in as <email> (<FREE | PRO>).
```

Then say in one sentence how to use it: ask a question about the library ("what do my books say about X?").

**On failure:**

| What you see | Tell the user |
|---|---|
| `401 Unauthorized`, "not authenticated", "invalid token" | Reconnect Trove in ChatGPT's app settings and sign in with the account used at heytrove.ai. |
| Timeout or 5xx | Trove is having a temporary problem; try again in a moment. Do not call it an auth problem or a plan limit. |

Never guess the account or plan — report only what `whoami` returned. If the user has no account yet, point them at https://heytrove.ai to sign up.
