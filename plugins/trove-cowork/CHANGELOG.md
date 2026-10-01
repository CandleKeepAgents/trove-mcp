# Changelog

## 1.1.1 — Directory listing metadata

- Set the directory display name to Trove and bundle the square listing icon inside the plugin.
- Declare privacy, support, documentation and terms URLs explicitly for the directory preview.
- Remove the outdated downloadable ZIP fallback and describe the current disconnect flow accurately, in the README and in the setup skill.

## 1.1.0 — Page-level writes, undo, delete, shelves, manuscripts

- The connector now has 38 tools (20 new). The skill's tool table maps each one to the agent that calls it.
- **book-writer** picks the smallest write: `append_item_pages` to add content (no version needed), `put_item_page` to replace one page (with `expectedVersion`), and `put_item_content` only to restructure. Manuscript entries are appended or merged into one page instead of rewriting the whole book.
- **book-writer** also handles undo (`list_item_versions`, `restore_item_version`), deletes (`delete_items`, only after the user confirms the exact books; `unsubscribe_marketplace` for marketplace books), shelves (`list_shelves`, `get_shelf`, `create_shelf`, `update_shelf`, `delete_shelf`, `add_to_shelf`, `remove_from_shelf`) and manuscripts (`list_manuscripts`, `create_manuscript`, `update_manuscript`, `archive_manuscript`). A preview book (`version: null` with `versionWithheld`) is reported as not editable.
- **item-reader** records an in-book gap with `report_gap { aboutItemId, aboutSection }` when a book it read with full access lacks the sub-topic the user needed (never on a Pro preview).
- **librarian** handles reader requests on the user's own books (`list_book_gaps`, `resolve_book_gap`, `restore_book_gap`).
- **item-reader** and **book-enricher** pass the `start_access_session` id as `sessionId` on every read, so reads are recorded under the research session.
- **book-enricher** can set the book's `outcomes` list.
- The skill has a library-management route table (undo, delete, shelves, manuscripts, reader requests).

## 1.0.0 — Trove

- First release under the Trove name. This plugin succeeds `candlekeep-cowork` 0.7.2; behaviour and workflow are unchanged.
- Plugin renamed to `trove-cowork`, skill to `trove`, setup skill to `trove-setup`. Sub-agents are addressed as `trove-cowork:<agent>`.
- The bundled connector is now the `trove` server at `https://heytrove.ai/api/v1/mcp`, so tools are called as `trove:<tool>`.
- All links point at `heytrove.ai`; support is `support@heytrove.ai`.
- The skill states once that CandleKeep is the former name of Trove, so a user who says "CandleKeep" still reaches it.
- The skill's tool inventory is a table of tool → calling agent, so a new server tool is one row.
- Plan limits are no longer quoted as numbers in the skill or README (they differ by pricing cohort); the agent relays the limit the server returns.
- Home repository is now `CandleKeepAgents/trove-mcp`.
