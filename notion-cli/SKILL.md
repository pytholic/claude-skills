---
name: notion-cli
description: Works with Notion pages and databases through the `ntn` CLI or the Notion MCP, picking the cheaper tool for each job. Use for any Notion task — reading, creating, editing, appending to, or trashing a page, querying or adding database rows, or uploading an image to a page.
---

# Notion: `ntn` CLI or Notion MCP

Default to the official `ntn` CLI. It adds nothing to context at startup, and `ntn pages get` returns Markdown at about a third of the tokens the MCP uses. Use the Notion MCP only where the CLI is clumsy: database-heavy work.

## Choose the tool

| Task | Use | Why |
|---|---|---|
| Read a page | `ntn pages get <id>` | Markdown output, ~3x fewer tokens |
| Create, replace, or trash a page | `ntn pages create` / `edit` / `trash` | Simple and cheap |
| Partial edit or append to a page | `ntn api v1/pages/<id>/markdown -X PATCH -d @body.json` | `pages edit` replaces the whole page |
| Create or update database rows, set properties | Notion MCP (`create-pages`, `update-page`) | One call sets row properties; `ntn` needs hand-built property JSON |
| Query a database | Notion MCP | Structured filters and results |
| Upload an image | Either; both are multi-step | See below |

## `ntn` rules

- Setup: `npm i -g ntn`, then `ntn login`. No `NOTION_API_TOKEN` is needed.
- Always pass `</dev/null` to `ntn api` calls; they hang on an open stdin.
- Write JSON request bodies to a file and pass `-d @file.json`, rather than inlining JSON.
- For partial page edits, the markdown PATCH body uses `update_content` (replace a section) or `insert_content` (append or insert).
- Avoid raw `ntn api` block reads and `--spec`; they return 9–13 KB of output.

## Images

Upload first, then reference the upload id:
- CLI: `ntn files create < img.png`, then reference the returned id.
- MCP: `create-file-upload`, `curl` the file to the returned URL, then reference the id.

## Quirk on both

A leading `# H1` in the body of a new page is swallowed into the page title. Start the body with the first real section instead.
