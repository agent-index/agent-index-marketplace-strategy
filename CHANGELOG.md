# Changelog

## 1.0.0

Initial release.

### Infrastructure Alignment — Remote Filesystem Migration

Aligned with agent-index v2.0.0 remote filesystem architecture. No functional changes to strategy workflows.

**Breaking:** Requires `agent_index_min_version: 2.0.0`. The agent-index-filesystem MCP server must be available for shared strategy operations.

- Updated `agent_index_min_version` in `collection.json` from `1.0.0` to `2.0.0`
- Added `reads_from` and `writes_to` paths to `share-strategy.md` (was `null` despite `produces_shared_artifacts: true`)
- Updated `collection-setup.md` prerequisites and parameter descriptions to reference remote filesystem and `aifs_*` MCP tools
- Added tool selection guidance to all task workflows: private workspace operations use native Read/Write tools; shared strategy operations use `aifs_*` MCP tools (`aifs_read`, `aifs_write`, `aifs_exists`)
- Updated `share-strategy.md` workflow steps with explicit `aifs_exists`, `aifs_read`, `aifs_write` references for shared space operations
- Updated manifest operations (`strategies-manifest.json`) across all relevant tasks to specify `aifs_read`/`aifs_write`
- Updated `README.md` sharing model section to reference remote filesystem access via `aifs_*` MCP tools
- Updated `strategy-tutorial.md` to reference "remote filesystem" instead of "filesystem"
