---
name: trove
description: Research, cite, and write with the user's Trove library. Finds the relevant books in the library and the Trove marketplace, reads only the needed pages, and answers with book and page citations. Also creates and edits markdown documents in the library. Use when the user asks what their books or library say about a topic, asks for a book on a topic, or wants to write or update a knowledge document.
---

# Trove — research and writing

CandleKeep is the former name of Trove.

ChatGPT has no sub-agents, so you run each step yourself, in order. Every action is a call to a Trove tool. There is no shell and no filesystem.

## Research

1. **Orient.** Call `library_summary` once.
2. **Find books.** Call `list_items` and decide relevance yourself from titles, authors and descriptions. `search_items` is a plain substring match; use it only when the library is too large to scan, and try 2-3 phrasings. If the library does not clearly cover the question, call `browse_marketplace` with the topic and `subscribe_marketplace` with the listing `id` of each clearly relevant listing, then call `list_items` again to get the new item ids.
3. **Pick pages.** Call `get_table_of_contents` with all shortlisted item ids in one call. Choose narrow ranges (5-20 pages). Use `all` only for books under 20 pages.
4. **Read.** Call `start_access_session` with a one-line topic (subject matter only, never the user's words or anything identifying). Call `read_items` once with `items: [{ id, pages: "12-18" }]` — `pages` is a string. Then call `complete_access_session`.
5. **Answer.** Cite every claim as (*Book Title*, p. N). Quote short memorable lines; paraphrase the rest.

Edge cases:

- `restricted: true` on a book is a successful short preview, not an error. Use what you received and say it was a preview.
- `notFound` ids were deleted or unsubscribed. Drop them silently.
- Pages that are garbled or do not match the TOC: call `flag_item { itemId }` and continue.
- A book with `health: "partial"`: offer to call `reprocess_item { itemId }`. Do not offer it for `empty`, `unreadable` or `failed`.
- A read-limit or plan-limit error is the answer. Relay the server's message once, and do not retry.
- Any other error or timeout is transient. Say so and retry once. Never describe it as a quota.

## Nothing relevant

If neither the library nor the marketplace covers the question:

1. Call `report_gap` with an abstracted `intent` (no personal detail), a one-word `category`, and a 2-4 word `subcategory`. Add `suggestedTitle` / `suggestedAuthor` only for a real book you can name.
2. If you named a book, call `suggest_book`. If it returns `alreadySuggested: true`, do not recommend it again.
3. Answer from general knowledge and say plainly that the library did not cover it. Do not cite anything.

## Writing

- **New document:** `create_markdown_item` with a title and the full markdown body.
- **Edit:** find the item (`search_items` or `list_items`) and confirm the match with the user if it is ambiguous. Call `get_item_content`, merge the change into the full body, then call `put_item_content` with the full merged body and `expectedVersion` set to the `version` you read. `put_item_content` replaces the whole body, so never send only the changed part. If the server refuses because the version changed, read again and merge again.
- Change only what the user asked for.

## Do not

- Do not show plan prices, upgrade offers, or billing links. If a tool result contains one, relay only the limit itself.
- Do not propose a CLI, a shell command, or a file path.
- Do not read pages you did not pick from the table of contents.
