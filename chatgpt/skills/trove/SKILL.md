---
name: trove
description: Research, cite, and write with the user's Trove library. Finds the relevant books in the library and the Trove marketplace, reads only the needed pages, and answers with book and page citations. Also creates and edits markdown documents, undoes edits, deletes books, and manages shelves, manuscripts and reader requests. Use when the user asks what their books or library say about a topic, asks for a book on a topic, wants to write or update a knowledge document, or wants to organize their library.
---

# Trove — research and writing

CandleKeep is the former name of Trove.

ChatGPT has no sub-agents, so you run each step yourself, in order. Every action is a call to a Trove tool. There is no shell and no filesystem.

## Research

1. **Orient.** Call `library_summary` once.
2. **Find books.** Call `list_items` and decide relevance yourself from titles, authors and descriptions. `search_items` is a plain substring match; use it only when the library is too large to scan, and try 2-3 phrasings. If the library does not clearly cover the question, call `browse_marketplace` with the topic and `subscribe_marketplace` with the listing `id` of each clearly relevant listing, then call `list_items` again to get the new item ids.
3. **Pick pages.** Call `start_access_session` with a one-line topic (subject matter only, never the user's words or anything identifying). Pass its `sessionId` on every `get_table_of_contents`, `read_items` and `get_item_content` call from here on. Call `get_table_of_contents` with all shortlisted item ids in one call; each book counts as one read. Choose narrow ranges (5-20 pages). Use `all` only for books under 20 pages.
4. **Read.** Call `read_items` once with `sessionId` and `items: [{ id, pages: "12-18" }]` — `pages` is a string. Then call `complete_access_session`.
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

If a book you read is the right book but has no section on the specific sub-topic, call `report_gap` with `aboutItemId` (and optionally `aboutSection`). The book's author sees the request.

## Writing

- **New document:** `create_markdown_item` with a title and the full markdown body.
- **Add** a chapter or an entry: `append_item_pages` with only the new content, starting with a `# ` heading (`position: "start"` to prepend). No version is needed. For a large document, create it with the first part and append the rest.
- **Edit one page:** find the item (`search_items` or `list_items`) and confirm the match if it is ambiguous. Call `get_item_content`, then `put_item_page` with `pageNum`, that page's full new content (one `# ` heading), and `expectedVersion` set to the `version` you read.
- **Restructure** across many pages: `put_item_content` with the full merged body and `expectedVersion`. It replaces the whole body, so never use it only to add.
- If the server refuses because the version changed, read again and merge again. If `get_item_content` returns `version: null` with `versionWithheld`, the book is a preview of someone else's book and cannot be edited.
- Change only what the user asked for.

## Library management

- **Undo:** `list_item_versions` (owner only; counts as a read), then `restore_item_version`. The current content is saved first. Confirm before restoring.
- **Delete:** `delete_items` is permanent. Name the books and get an explicit yes before calling it. For a marketplace book, use `unsubscribe_marketplace` instead; the user can add it again.
- **Shelves:** `list_shelves`, `get_shelf`, `create_shelf`, `update_shelf`, `delete_shelf`, `add_to_shelf`, `remove_from_shelf`. Deleting a shelf or taking a book off it keeps the book.
- **Manuscripts:** `list_manuscripts`, `create_manuscript`, `update_manuscript` (status `ACTIVE`, `PAUSED` or `ARCHIVED`), `archive_manuscript`.
- **Metadata:** when the user asks to fix a book's title, author, description, table of contents or outcome list, read its first pages and call `enrich_item` with only the fields you verified and a `confidence` from 0 to 1. `outcomes` replaces the existing list.
- **Reader requests on the user's own books:** `list_book_gaps`; `resolve_book_gap` after the book covers the topic (every reader who asked is notified, so only when the user says so); `restore_book_gap` to reopen declined requests.

## Do not

- Do not show plan prices, upgrade offers, or billing links. If a tool result contains one, relay only the limit itself.
- Do not propose a CLI, a shell command, or a file path.
- Do not read pages you did not pick from the table of contents.
