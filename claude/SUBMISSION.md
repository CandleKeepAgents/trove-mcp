# Submit Trove to Anthropic's directory

This checklist is for the person who submits Trove to Anthropic. Items marked **TODO** need a human.

Requirements are from the Claude documentation, read on 2026-09-30:

- Submit a plugin: <https://claude.com/docs/plugins/submit>
- Plugin pre-submission checklist: <https://claude.com/docs/plugins/pre-submission-checklist>
- Submit a connector: <https://claude.com/docs/connectors/building/submission>
- Connector review criteria: <https://claude.com/docs/connectors/building/review-criteria>
- Connector authentication: <https://claude.com/docs/connectors/building/authentication>

Read them again before you submit.

## 1. Two submissions

The portal is now at <https://claude.ai/directory/manage>. It replaces the earlier Console form (`platform.claude.com/plugins/submit`). If an earlier submission exists in the Console form, move it to the portal as the documentation tells.

Make two submissions:

| Submission | Portal choice | Source |
|---|---|---|
| The plugin (skill, 4 sub-agents, connector) | **Plugin bundle** | Repository `CandleKeepAgents/trove-mcp`, plugin path `plugins/trove-cowork`, branch `main` |
| The MCP server | **MCP connector** | `https://heytrove.ai/api/v1/mcp` |

Anthropic tells you to submit the server as a connector also, when your plugin uses a server that you operate. The connector listing gives you the server details, the authentication settings and the dashboard, and it lets you pair the server with the plugin.

- [ ] **TODO:** The submitter has a paid Claude plan and a role that can submit.
- [ ] **TODO:** Connect the GitHub account of the submitter on claude.ai, in the same Claude organization. That account must have push access to `CandleKeepAgents/trove-mcp`.
- [ ] **TODO:** If the old plugin (formerly called CandleKeep) has a live listing, decide if you delist it after the Trove listing is live.

## 2. Plugin bundle checks

The CI workflow `.github/workflows/validate.yml` does the local checks. The portal **Validate** button does more checks.

| Requirement | Status |
|---|---|
| Public GitHub repository | Yes: `CandleKeepAgents/trove-mcp` is public. |
| `claude plugin validate` passes | Yes: `claude plugin validate --strict ./plugins/trove-cowork` and `claude plugin validate --strict .` pass (Claude Code 2.1.285). CI runs both. |
| `.claude-plugin/plugin.json` in the plugin folder | Yes |
| `name` is lowercase, hyphens, 64 characters or fewer, not a reserved word | Yes: `trove-cowork` |
| `description`, `author`, `version` are set | Yes |
| README of 40 words or more in the plugin folder | Yes: `plugins/trove-cowork/README.md` |
| LICENSE in the plugin folder | Yes: MIT, holder Trove |
| Consistent versions | Yes: `plugin.json`, `marketplace.json`, the CHANGELOG top entry and `chatgpt/plugin.json` are all the same version (now 1.1.1). CI fails if they differ. |
| Remote MCP server has `type: http` and an `https://` URL | Yes |
| No `.DS_Store` or other OS files | CI fails if one is present. |
| No hooks, scripts, launchers or lockfiles | Yes. The plugin is only Markdown and JSON, so nothing is held for a reviewer for this reason. |
| Skill and agent files have YAML front matter with a text `description` | Yes |

Raise `version` in `plugin.json` and `marketplace.json`, and add a CHANGELOG entry, for each release. The directory follows the tracked branch and scans each new commit.

Name risk: the portal holds a name that is only generic words. "Trove" is a dictionary word, so a reviewer can hold the plugin. This is a hold, not a rejection. The name stays "Trove" (Brand Book ch. 02: no descriptors in the name); put "knowledge library" in the one-liner.

## 3. Connector checks

- [x] **Tool annotations — done.** Each tool must have a `title` and the applicable `readOnlyHint` or `destructiveHint`. The server sends a `title` and all four hints on each of its 38 tools. The values are in `chatgpt/SUBMISSION.md`, section 5. In Claude, read-only tools run without a confirmation for each call, and destructive tools always ask.
- [ ] **TODO — tool descriptions.** A description must tell what the tool does. It must not tell Claude how to behave, and it must not promote a product. Examine these:
  - `library_summary` no longer says "Call this first on every research task". Done.
  - Tool results that contain "Upgrade to Professional" text and `upgradeUrl`. Anthropic prohibits promotion in tool descriptions. Keep upgrade text to plain limit messages. For ChatGPT the server already sends neutral limit wording with no upgrade link.
- [ ] Tool names are 64 characters or fewer. Yes.
- [ ] Tool results must be less than approximately 150,000 characters (claude.ai, Desktop, Cowork) and 25,000 tokens (Claude Code, default). A tool call must end in 240 seconds. Source: <https://claude.com/docs/connectors/building>. Examine `list_items`, `read_items` and `get_item_content` on a large library.
- [ ] Transport is Streamable HTTP. Yes.
- [ ] Every tool returns a useful error, not only "Internal Server Error". Check each tool with the MCP Inspector.
- [ ] **TODO — test in Claude.** Add `https://heytrove.ai/api/v1/mcp` as a custom connector and call each tool. The portal asks you to confirm this.

## 4. Authentication

Claude supports OAuth with dynamic client registration (DCR) by default. Trove has DCR at `https://heytrove.ai/api/oauth/register` and PKCE with `S256`. In the portal **Authentication** step, select OAuth with dynamic client registration.

| Requirement | Status |
|---|---|
| Unauthenticated request gets `401` with `WWW-Authenticate: Bearer resource_metadata="…"` | Present |
| `resource` in the protected resource metadata is exactly the MCP URL | Done: `OAUTH_ISSUER_URL=https://heytrove.ai` is set in production. |
| First entry of `authorization_servers` is the issuer | Present |
| Redirect URI `https://claude.ai/api/mcp/auth_callback` accepted | Present (DCR accepts HTTPS redirect URIs) |
| Claude Code loopback `http://localhost:<any port>/callback` and `http://127.0.0.1:<any port>/callback` | **TODO:** check that the authorize step does not compare the port. DCR accepts these URIs, but the port changes for each Claude Code session. |
| Token endpoint accepts `application/x-www-form-urlencoded` | **TODO:** check. |
| Refresh tokens rotate, and an invalid refresh token gets `invalid_grant` | **TODO:** check. |
| Discovery, registration and token endpoints answer in less than 10 seconds | **TODO:** check. |
| Traffic from `160.79.104.0/21` is not blocked by a WAF | **TODO:** check. |

Note: with DCR, Claude registers a new client for each new connection. If the directory brings much traffic, add Client ID Metadata Document support. Advertise `"client_id_metadata_document_supported": true` and `"none"` in `token_endpoint_auth_methods_supported`.

## 5. Listing materials

Connector listing (portal **Listing** step):

| Field | Value |
|---|---|
| Server name (100 characters) | Trove |
| One-liner (200 characters) | Your AI's knowledge library: search, read and cite your books to the page, add marketplace books, and write knowledge documents. |
| Description (2000 characters) | **TODO:** write it. Start from `plugins/trove-cowork/README.md`. Anthropic cannot edit it after you submit. |
| Categories (1 to 5) | **TODO:** for example Knowledge, Productivity, Research |
| Documentation URL | https://heytrove.ai/install?surface=cowork |
| Support page | https://heytrove.ai/support (public, no sign-in; monorepo PR 510) |
| Privacy policy URL | https://heytrove.ai/privacy |
| Support contact | support@heytrove.ai |
| Icon | `claude/assets/icon.png`: a square PNG of 512 x 512 px, the Trove mark in ink (`#0e2321`) on paper (`#fbf7ee`). |
| URL slug | **TODO:** `trove`. It is permanent after publication. |
| Reads or writes | Both |

The connector has no interactive UI (it is not an MCP App), so it does not need carousel screenshots.

## 6. Test account for reviewers

- [ ] **TODO:** Make a dedicated, fully populated review account on heytrove.ai. Enter its credentials only in the portal **Test & launch** step. Do not put them in the repository.
- [ ] Give it a Pro plan, so that no example stops at a limit.
- [ ] Put this data in it:
  - The **OWASP Web Security Testing Guide v4.2**, added from the Trove marketplace (example 1).
  - A PDF renamed `scan_001.pdf`, with no title and no author (example 4).
  - No document called "On-Call Escalation Policy", because example 3 creates it.
  - Do not add **Prisma ORM 7 Complete Reference** before the review, because example 2 adds it. Do not add **Kubernetes: Official Documentation** either (a ChatGPT test case adds it).
- [ ] Write the sign-in steps so that a reviewer can connect with no help.
- [ ] Note for the sign-in steps: until the rebrand flip, sign-in on heytrove.ai goes through getcandlekeep.com, the domain of CandleKeep, the former name of Trove (heytrove.ai is a Clerk satellite of it). The reviewer sees a getcandlekeep.com sign-in page and then returns to heytrove.ai. Say this in the sign-in steps, so that the reviewer does not think that the redirect is an error.

## 7. Example prompts (3 or more)

These are in the plugin README. Run each with the review account before you submit.

1. "What does my library say about testing for SQL injection? Cite the pages." Expected: the answer cites the OWASP Web Security Testing Guide v4.2 with page numbers.
2. "I need to get up to speed on Prisma. Check the Trove marketplace, add the best book, and summarise how it handles migrations." Expected: adds "Prisma ORM 7 Complete Reference" and cites it.
3. "Draft a knowledge doc called "On-Call Escalation Policy" covering our rotation and severity levels. Then add a section on paging escalation."
4. "My library has a book called scan_001.pdf with no author. Clean up its metadata." Expected: the enricher sets a real title and author.

## 8. Data handling and compliance answers

| Question | Answer |
|---|---|
| Reads or stores personal data | Yes. It reads the library of the connected account. It stores tool calls and three short agent-written texts: a research topic, a gap topic, a book-suggestion reason. The server removes emails, keys, tokens and phone numbers from these texts. |
| Sends data to other services | No. The plugin sends data only to its declared connector, `https://heytrove.ai/api/v1/mcp`. The plugin README discloses this destination and the data sent. |
| Retention | Done: the retention periods are in the "Data Retention" section of <https://heytrove.ai/privacy> (monorepo PR 510). |
| For people under 18 | **TODO:** decide. |
| API ownership | First-party API |
| Financial transactions | None. `subscribe_marketplace` adds a book to the library. It takes no payment and changes no plan. |
| Contact email | support@heytrove.ai (receives mail) |

## 9. After publication

- The directory scans each new commit on `main`. Set up the GitHub push webhook in the portal, so that the scan starts at each push.
- Email `mcp-review@anthropic.com` for connector escalations. Email `directory@anthropic.com` if another organization has submitted this repository.
