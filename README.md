# Trove MCP

Trove gives your AI assistant a library that it can read. You upload PDFs, ebooks and markdown documents to [heytrove.ai](https://heytrove.ai). Then Claude or ChatGPT can search them, read the necessary pages, and answer with the book and the page for each claim. The assistant can also add books from the Trove marketplace to your library, and write knowledge documents in your library.

CandleKeep is the former name of Trove.

This repository contains:

| Folder | Contents |
|---|---|
| `plugins/trove-cowork/` | The Claude plugin: the `trove` skill, four sub-agents, and the Trove connector |
| `.claude-plugin/marketplace.json` | The Claude plugin marketplace with the name `trove` |
| `chatgpt/` | The ChatGPT app package and its submission checklist |
| `claude/` | The submission checklist for Anthropic's directory |

One repository serves both stores. Anthropic reads a plugin from a folder in a GitHub repository, and a repository can hold more than one plugin folder. The Anthropic connector listing needs only the server URL. OpenAI takes a ZIP file that you upload, and it does not read a repository. The two stores use different manifest files, and each set is in its own folder, so they do not conflict.

## Install in Claude (Cowork, Claude Desktop, claude.ai)

1. Open **Customize → Plugins → Add marketplace**.
2. Enter `CandleKeepAgents/trove-mcp`.
3. Find **Trove** in the plugin list and click **Install**.
4. Sign in with your heytrove.ai account and click **Authorize**.

To use only the connector, without the skill and the sub-agents, add a custom connector with this URL:

```
https://heytrove.ai/api/v1/mcp
```

## Install in ChatGPT

1. Open the ChatGPT app directory and find **Trove**.
2. Click **Connect**.
3. Sign in with your heytrove.ai account and click **Authorize**.

## Tools

The connector uses OAuth 2.1. It asks for these scopes: `library:read`, `library:write`, `marketplace:read`, `marketplace:write`.

Each tool tells the assistant if it only reads, if it can remove or overwrite data, and if it reaches public content. Tools that can remove or overwrite data ask you before they run.

**Read**

| Tool | What it does |
|---|---|
| `whoami` | Shows the signed-in account and plan |
| `library_summary` | Shows the plan, the item count, recent items and active manuscripts |
| `list_items` | Lists all items in the library |
| `search_items` | Finds items by title, description or author |
| `get_table_of_contents` | Gets the table of contents of one or more items |
| `read_items` | Reads page ranges from one or more items |
| `get_item_content` | Gets the full markdown of an item and its version |
| `list_item_versions` | Lists earlier versions of a book that you own |
| `list_shelves` | Lists your shelves |
| `get_shelf` | Gets one shelf and its books |
| `list_manuscripts` | Lists your manuscripts |
| `list_book_gaps` | Lists the topics that readers asked for in books that you wrote |
| `browse_marketplace` | Lists books on the Trove marketplace |

**Write**

| Tool | What it does |
|---|---|
| `create_markdown_item` | Creates a markdown document |
| `append_item_pages` | Adds content at the end or the start of a document |
| `put_item_page` | Replaces one page of a document |
| `put_item_content` | Replaces the full markdown of a document |
| `restore_item_version` | Restores an earlier version. The current version is saved first. |
| `delete_items` | Deletes books that you own. This cannot be undone. |
| `enrich_item` | Sets a better title, author, description, table of contents or outcome list |
| `flag_item` | Marks an item for new metadata |
| `reprocess_item` | Imports a book again from its original file |
| `create_shelf`, `update_shelf`, `delete_shelf` | Creates, changes or deletes a shelf. Deleting a shelf keeps its books. |
| `add_to_shelf`, `remove_from_shelf` | Puts books on a shelf or takes them off |
| `create_manuscript`, `update_manuscript`, `archive_manuscript` | Creates, changes, pauses or archives a manuscript |
| `subscribe_marketplace` | Adds a marketplace book to the library (no payment) |
| `unsubscribe_marketplace` | Removes a marketplace book from the library. You can add it again. |
| `start_access_session`, `complete_access_session` | Starts and ends a reading session |
| `report_gap` | Records a topic that the library, or one book, does not cover |
| `suggest_book` | Records a suggested book, so that it is not suggested again |
| `resolve_book_gap` | Marks reader requests on your book as done and tells those readers |
| `restore_book_gap` | Opens declined reader requests on your book again |

## Example prompts

- "What does my library say about backpressure in async Rust? Cite the pages."
- "Find a marketplace book on Verilog, add it to my library, and summarize its approach to testbenches."
- "Create a doc in my library called "On-Call Escalation Policy" covering rotation and severity levels."
- "My library has a book called `document.pdf` with no author. Clean up its metadata."

## Privacy, terms and support

- Privacy policy: <https://heytrove.ai/privacy>
- Terms: <https://heytrove.ai/terms>
- Support: <https://heytrove.ai/support> or support@heytrove.ai

Trove does not keep a transcript of your conversation. It keeps the tool calls and three short texts that the assistant writes: a research topic, a topic that the library does not cover, and the reason for a book suggestion. The server removes emails, keys, tokens and phone numbers from these texts before it stores them. The plugin README gives more data.

## License

MIT. See [LICENSE](./LICENSE).
