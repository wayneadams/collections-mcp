# collections-mcp

Search-only MCP over an xAI Collection of a workspace's documents, plus an offline sync that keeps that index current.

The design is in [docs/design.md](docs/design.md). A rendered copy is [docs/design.html](docs/design.html). Status is draft. The package is not implemented yet, and it is not on PyPI.

## Two processes, two keys

| Process | Holds | Job |
| --- | --- | --- |
| `collections-mcp` | Inference key (`XAI_API_KEY`) | stdio MCP. Search, list, and get. |
| `collections-sync` | Management key, and the inference key for file upload | Walk the workspace, upload, update, and delete. |

Search calls `POST https://api.x.ai/v1/documents/search`. Sync talks to `https://management-api.x.ai`. The MCP process never uploads, never deletes, and never sees the management key.

Grok Build, Cursor, and Claude Code get the same four tools:

- `collections_search` — hybrid search over durable prose: READMEs, design docs, RFCs, runbooks, PDFs, office files. Source lookup stays with grep.
- `list_collections` — names and ids from the local catalog.
- `list_documents` — indexed files from the local catalog.
- `get_document` — metadata for one file, with an optional short local preview.

List and get read a local SQLite catalog written by sync. They do not call the Management API, and they do not download file bytes from xAI.

## Install

Python 3.12. Requires [uv](https://docs.astral.sh/uv/).

Until the package is on PyPI, the command is:

```text
uvx --from git+https://github.com/wayneadams/collections-mcp collections-mcp
```

That starts working once the package is in this repository. Right now the repo holds the design.

```json
{
  "mcpServers": {
    "collections": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/wayneadams/collections-mcp",
        "collections-mcp"
      ],
      "env": {
        "XAI_API_KEY": "<inference key>",
        "COLLECTIONS_MANIFEST": "/path/to/collections.yaml"
      }
    }
  }
}
```

Leave the management key out of that `env` block. Sync reads both keys from `collections.yaml`, which is gitignored.

Grok Build, once the package exists:

```text
grok mcp add collections --env XAI_API_KEY=<inference key> --env COLLECTIONS_MANIFEST=/path/to/collections.yaml -- uvx --from git+https://github.com/wayneadams/collections-mcp collections-mcp
```

Cursor and Claude Code use the same `mcpServers` block.

## License

MIT.
