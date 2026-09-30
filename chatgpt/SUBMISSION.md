# Submit Trove to the ChatGPT app directory

This checklist is for the person who submits Trove to OpenAI. Do the steps in sequence. Items marked **TODO** need a human.

Requirements are from the OpenAI developer documentation, read on 2026-09-30:

- Submission: <https://developers.openai.com/apps-sdk/deploy/submission>
- Guidelines: <https://developers.openai.com/apps-sdk/app-submission-guidelines>
- Authentication: <https://developers.openai.com/apps-sdk/build/auth>

Read them again before you submit. OpenAI changes these pages.

## 1. What you submit

OpenAI accepts a ZIP file of an "Agent Plugins" package. You upload it at <https://platform.openai.com/plugins>. This folder is that package:

| File | Purpose |
|---|---|
| `plugin.json` | Package identity, the listing (`extensions.com.openai.interface`), the review test cases, and the release notes |
| `mcp.json` | The one MCP server: `trove` at `https://heytrove.ai/api/v1/mcp` |
| `skills/get-started/SKILL.md` | Onboarding skill (`onboardingSkill`). It calls `whoami`. |
| `skills/trove/SKILL.md` | The research and writing workflow |
| `assets/` | Icons and logos. **TODO:** add them (see step 4). |

Make the ZIP from inside this folder, so that `plugin.json` is at the root of the ZIP:

```bash
cd chatgpt && zip -r ../trove-chatgpt.zip plugin.json mcp.json skills assets -x '*.DS_Store'
```

Do not put credentials in the ZIP. You enter the test account in the dashboard.

### Skills, but no sub-agents

The OpenAI package format discovers skills in `skills/`. It has no sub-agents. Thus the Claude workflow (librarian, item-reader, book-writer, book-enricher) is one skill here, and ChatGPT does all steps in one conversation. The MCP server tool descriptions must also carry the important rules (narrow page ranges, `pages` is a string, `expectedVersion` on writes), because a client can ignore a skill.

## 2. Organization and identity

- [ ] **TODO:** The submitter is an owner of the OpenAI organization, or has the **Apps Management Write** permission.
- [ ] **TODO:** Complete business verification for the organization in the OpenAI Platform settings. The directory shows the verified name.

## 3. Listing values (already in `plugin.json`)

| Field | Value | Limit |
|---|---|---|
| `displayName` | Trove | 30 characters |
| `shortDescription` | Read and cite your own books | 30 characters |
| `longDescription` | In `plugin.json` | 4000 characters |
| `developerName` | Trove | 80 characters |
| `category` | Productivity. **TODO:** confirm in the dashboard list. | Dashboard list |
| `websiteURL` | https://heytrove.ai | HTTPS |
| `supportURL` | https://heytrove.ai/support | HTTPS |
| `privacyPolicyURL` | https://heytrove.ai/privacy | HTTPS |
| `termsOfServiceURL` | https://heytrove.ai/terms | HTTPS |
| `defaultPrompt` | 3 prompts in `plugin.json` | 3 prompts, 128 characters each |

- [ ] **TODO:** Make sure that `https://heytrove.ai/support` opens without sign-in. Today the support page is in the signed-in dashboard. OpenAI requires that each URL is accessible and names the same publisher.
- [ ] **TODO:** Make sure that the privacy policy states the categories of personal data, the purposes, the categories of recipients, and the retention periods. OpenAI requires all four.
- [ ] Note: the guidelines tell you not to use a single dictionary word as the name. "Trove" is a dictionary word. If the review rejects it, use "Trove Library" in `displayName`.

## 4. Images

- [ ] **TODO:** Add these files to `assets/`: `icon.png`, `icon-dark.png`, `logo.png`, `logo-dark.png`. Use PNG, JPEG, WebP or SVG. Each image must be square, 48 px to 4096 px, and 5 MiB or less. `composerIcon` and `logo` are necessary. The dark versions are optional.
- [ ] Optional: add `brandColor` and `brandColorDark` to `interface`.
- OpenAI does not show screenshots in the directory now. Do not make them.

## 5. MCP server

Server URL: `https://heytrove.ai/api/v1/mcp` (streamable HTTP).

- [ ] **TODO — domain verification.** OpenAI gives a challenge token. Serve it at `https://heytrove.ai/.well-known/openai-apps-challenge`. The webapp has no route for this path now. Add one that returns the token from an environment variable.
- [x] **Tool annotations — done.** OpenAI requires three explicit boolean values on each tool: `readOnlyHint`, `destructiveHint`, `openWorldHint`. The server sends all four hints (with `idempotentHint`) on each of its 38 tools. Values as the server sends them ("new" marks the 20 tools added in 1.1.0). When the server gets a new tool, add a row here:

| Tool | readOnlyHint | idempotentHint | destructiveHint | openWorldHint | |
|---|---|---|---|---|---|
| `whoami` | true | true | false | false |  |
| `library_summary` | true | true | false | false |  |
| `list_items` | true | true | false | false |  |
| `search_items` | true | true | false | false |  |
| `get_table_of_contents` | true | true | false | false |  |
| `read_items` | true | true | false | false |  |
| `get_item_content` | true | true | false | false |  |
| `list_item_versions` | true | true | false | false | new |
| `list_shelves` | true | true | false | false | new |
| `get_shelf` | true | true | false | false | new |
| `list_manuscripts` | true | true | false | false | new |
| `list_book_gaps` | true | true | false | false | new |
| `browse_marketplace` | true | true | false | true |  |
| `create_markdown_item` | false | false | false | false |  |
| `append_item_pages` | false | false | false | false | new |
| `create_shelf` | false | false | false | false | new |
| `create_manuscript` | false | false | false | false | new |
| `start_access_session` | false | false | false | false |  |
| `report_gap` | false | false | false | false |  |
| `flag_item` | false | true | false | false |  |
| `update_shelf` | false | true | false | false | new |
| `add_to_shelf` | false | true | false | false | new |
| `update_manuscript` | false | true | false | false | new |
| `resolve_book_gap` | false | true | false | false | new |
| `restore_book_gap` | false | true | false | false | new |
| `complete_access_session` | false | true | false | false |  |
| `subscribe_marketplace` | false | true | false | false |  |
| `suggest_book` | false | true | false | false |  |
| `put_item_content` | false | false | true | false |  |
| `reprocess_item` | false | false | true | false |  |
| `unsubscribe_marketplace` | false | false | true | false | new |
| `enrich_item` | false | true | true | false |  |
| `put_item_page` | false | true | true | false | new |
| `restore_item_version` | false | true | true | false | new |
| `delete_items` | false | true | true | false | new |
| `delete_shelf` | false | true | true | false | new |
| `remove_from_shelf` | false | true | true | false | new |
| `archive_manuscript` | false | true | true | false | new |

- [ ] **Result shape.** Each tool must return `structuredContent` (concise data for the model) and `content`. Data that only the client uses goes in `_meta`. The server sends `structuredContent` and `content` now. OpenAI also lists an output schema for each tool that returns structured data; the server declares none. **TODO:** add `outputSchema` to each tool, and make sure that each result agrees with it. Source: <https://developers.openai.com/apps-sdk/build/mcp-server>.
- [ ] UI widgets are optional. Trove submits as a data-only app, so it needs no `_meta.ui.resourceUri`, no HTML resource and no CSP.
- [x] **Server instructions — done.** The server sends an `instructions` string (approximately 1,600 characters). OpenAI recommends 512 characters or fewer; this is a recommendation, not a check. Keep the main workflow in the first 500 characters, because a client can shorten the text. Tool descriptions must describe what the tool does. They must not tell the model to call other tools or to promote a product.
- [ ] **TODO — upgrade text in tool results.** The guidelines prohibit plan offers, upgrade prompts, and links to checkout pages in the app. Tool results now contain `upgradeUrl` (`/billing`) and "Upgrade to Professional" text. Remove these when the client is ChatGPT. You can identify the client from the OAuth client that ChatGPT registers, or from `clientInfo.name` in `initialize`. A link to an informational plans page is permitted.
- [ ] Check that no tool result contains session IDs, trace IDs or diagnostic metadata that the user did not ask for. `start_access_session` must return its `sessionId`, because `complete_access_session` needs it.

## 6. Authentication (OAuth 2.1)

Trove already has an OAuth server. The values below come from the webapp code.

| Item | Value | Status |
|---|---|---|
| Protected resource metadata | `https://heytrove.ai/.well-known/oauth-protected-resource` (also `/.well-known/oauth-protected-resource/api/v1/mcp`) | Present |
| Authorization server metadata | `https://heytrove.ai/.well-known/oauth-authorization-server` | Present |
| Authorization endpoint | `https://heytrove.ai/oauth/authorize` | Present |
| Token endpoint | `https://heytrove.ai/api/oauth/token` | Present |
| Dynamic client registration | `https://heytrove.ai/api/oauth/register` | Present. Accepts any HTTPS redirect URI. |
| Revocation | `https://heytrove.ai/api/oauth/revoke` | Present |
| PKCE | `code_challenge_methods_supported: ["S256"]` | Present. OpenAI refuses servers without it. |
| Scopes | `library:read`, `library:write`, `marketplace:read`, `marketplace:write` | Present |
| Token audience | Tokens are bound to the MCP resource (RFC 8707) | Present |
| Client ID Metadata Document | Not supported | Optional. OpenAI prefers it to DCR. |
| Issuer identification (RFC 9207) | Not advertised | Optional. With it, ChatGPT uses one stable redirect URI. |

ChatGPT redirect URIs:

- `https://chatgpt.com/connector_platform_oauth_redirect` if the server supports issuer identification.
- `https://chatgpt.com/connector/oauth/{callback_id}` if it does not. Copy the exact value from the MCP server page in the dashboard.

DCR accepts both, so no allowlist change is necessary.

- [ ] **TODO — issuer.** Set `OAUTH_ISSUER_URL=https://heytrove.ai` in production. If it is not set, the code uses the old domain of the product. Then the `resource` in the metadata does not agree with the MCP URL, and the connection fails.
- [ ] **TODO — no redirect on the MCP URL.** Make sure that `https://heytrove.ai/api/v1/mcp` answers directly. A redirect from the apex domain to `www` breaks MCP clients.
- [ ] Check with `curl -i https://heytrove.ai/api/v1/mcp`: the answer must be `401` with a `WWW-Authenticate` header that points to the protected resource metadata.

## 7. Test account for reviewers

- [ ] **TODO:** Make a dedicated review account on heytrove.ai with sample data only. Do not use a real user's account.
- [ ] The account must sign in with an email and a password. OpenAI does not accept MFA, email or SMS codes, magic links, or a private network. Make sure that the Clerk sign-in for this account does not ask for an email code.
- [ ] Give the account a Pro plan, so that no test case stops at a limit.
- [ ] Put these books in the account, because the test cases use them: one book about incident response with severity levels, and at least one Rust async book on the marketplace. The account must not contain a document called "On-Call Escalation Policy", because a test case creates it.
- [ ] Keep the account and its data for later reviews.
- [ ] Enter the login URL, the email, the password and the sign-in steps in the dashboard **Review details** form.

## 8. Test cases and demo recording

`plugin.json` contains 5 positive test cases and 3 negative test cases. OpenAI requires at least these numbers.

- [ ] Run all 8 cases with the review account in ChatGPT developer mode before you submit. Correct `tools_triggered` if the real calls are different.
- [ ] **TODO:** Record a video that shows the test cases. Put its URL in `review.demo_recording_url` (now a placeholder).
- [ ] Complete the policy attestations in the dashboard.
- [ ] `commerce` is `false`. Trove sells nothing through ChatGPT.

## 9. Submit

1. Open <https://platform.openai.com/plugins>.
2. Upload the ZIP. Connect the MCP server and complete domain verification.
3. Correct all findings on the **Metadata & Skills** tab and the **MCPs** tab.
4. Submit. OpenAI sends the decision by email. Only one review can be active at a time.

After publication, OpenAI scans the tools each day. A new tool is examined without a new submission. To change the MCP server URL, you must contact OpenAI support.
