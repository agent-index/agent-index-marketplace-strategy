# Changelog

## [1.1.3] — 2026-06-05

### Changed

- **Hard gate amended (share-strategy 1.1.3, edit-strategy 1.1.2):** pointer writes proceed on the
  outcome file reporting `applied` OR — when the file is missing or carries a page-lifecycle value
  (`terminated`/`page_closed`) despite a confirmed Accept — an independent `aifs_get_permissions`
  read confirming every requested grant. Encodes the §3-C finding (bug `20260605-62a14c43`): helper
  ≤0.4.0 raced page-close when writing outcome files; 0.4.1 fixes the race (outcome persisted the
  moment apply completes), and the verification fallback is retained as defense in depth.

---

## [1.1.2] — 2026-06-04

### Changed (prose only — companion to core 3.9.0 My Drive member spaces)

- Member space is now the owner's `Agent-Index-Private` folder in **their own My Drive** (core
  3.9.0): share-strategy About/Step 1 updated — the owner OWNS the folder, which is what makes
  owner-applied grants possible (closes finding **F12**: Shared-Drive folders can only be shared by
  drive Managers, so 1.1.0/1.1.1 sharing failed at the owner's Accept with "insufficient
  permissions").
- `member_folder_id` self-provisions (`@ai:update` Migration 2 / `@ai:member-bootstrap`); the
  admin-backfill halt message is retired.
- Soft-delete nuance: pointers (Shared Drive) always overwrite-never-delete; content in the owner's
  My Drive is owner-deletable but archive-marking is preferred.
- No mechanics changes: anchors, bare-ID helper specs, outcome-gated pointers all as in 1.1.1.
  **Requires agent-index-core 3.9.0+** (`agent_index_min_version` bumped). share-strategy → 1.1.2,
  create-strategy → 1.1.1.

---

## [1.1.1] — 2026-06-04

### Fixed (from test-plan §3 live run on testproduction)

- **Helper specs now use the bare `id:{folder_id}` resource form** (the exact strategy folder's Drive ID, captured via `aifs_stat` at share time) instead of `id:{member_folder_id}/strategies/{slug}/`. permission-helper-go ≤0.3.0 rejected ALL id-anchor resources (`resource must be a string starting with "/"`), so v1.1.0 sharing could never apply a grant; 0.4.0 accepts the bare-ID form only. Granting the precise folder is also the least-privilege surface. **Requires permission-helper-go 0.4.0+.**
- **Pointer writes hard-gated on the helper outcome file** reporting `"outcome": "applied"` (share-strategy step 4→5; edit-strategy access changes). The §3 run wrote pointers on assumed success while every grant had been rejected — leaving discovery records advertising access that didn't exist. `partial_failure` now writes the pointer with only the applied grants.

---

## [1.1.0] — 2026-06-03

### Changed — owned-content sharing model (replaces the shared-directory model)

- **Shared strategy content now lives in the owner's private member space**, not under `/shared`. `share-strategy` copies the local strategy to `id:{member_folder_id}/strategies/{slug}/` (ID-anchor addressing, adapter 2.5.0+) and applies **per-person grants** via `permission-change-helper` (owner Accepts): *share with X* = X reader; *make X a collaborator* = X writer; *share with the org* = `all@{domain}` reader on that one strategy folder. Nobody else can see it. (Fixes the non-admin share/collaborate failure from the cross-collection access audit, same class as `20260531-8d20ea22`, with least-privilege instead of an org-wide writer grant.)
- **Discovery via pointer index.** Each shared strategy writes one pointer JSON to `/shared/strategies-index/{owner_hash}-{slug}.json` (type/owner/owner_hash/slug/folder_id/scope/title/shared_date). Tasks open shared strategies at `id:{folder_id}/...` from the pointer. The old `{shared_strategies_path}` directory and `strategies-manifest.json` are **retired** — no shared manifest exists in 1.1.0+ (this also removes the concurrent-manifest-write risk).
- **Soft-delete/unshare conventions** (core standards): members can't trash on Drive; unshare = revoke grants via helper + overwrite pointer `scope` to `"revoked"`; archive = mark `status: archived` in the strategy frontmatter. Never delete pointers or remote files.
- All capability tasks (`create-strategy`, `edit-strategy`, `share-strategy`, `run-briefing`, `strategy-review`, `manage-sources`, `manage-opportunities`) reworked: anchor-based tool selection (local private → native tools; own shared → `id:{member_folder_id}/...`; shared-with-you → pointer index → `id:{folder_id}/...`), manifest-update steps removed, creation is always private (share-right-away hands off to `share-strategy`).

### Added

- **`collaborative-acls.json`** (collection root) — single grant: `all@{domain}` **writer** on `/shared/strategies-index/` (the pointer index only — never strategy content), provisioned at install via `install-collection` Step 5.5.
- `setup/collection-setup.md` Setup Completion now creates `/shared/strategies-index/` (placeholder README) so the Step 5.5 grant can apply.

### Removed

- `shared_strategies_path` setup parameter and `strategies-manifest.json` initialization. Upgrading admins: delete the `shared_strategies_path` response; previously shared strategies should be re-shared by their owners (content relocates to their member space); the old `/shared/strategies/` directory can then be archived.

### Requirements / Notes

- **Requires agent-index-core 3.8.0+** (`member_folder_id` in members-registry/member-index) and **filesystem adapter 2.5.0+** (ID anchors, `stat.id`). Ships as part of the coordinated core 3.8.0 / adapter 2.5.0 release.
- **Admin actions after upgrade:** run the upgrade interview (collection-setup) → ensure `/shared/strategies-index/` exists → `install-collection` Step 5.5 applies the index grant (interactive helper Accept). Backfill `member_folder_id` for existing members per core 3.8.0 CHANGELOG.
- Capability versions: all seven tasks + the tutorial skill 1.0.2 → 1.1.0; `collection.json` 1.0.3 → 1.1.0; manifests reconcile on members' next `apply-updates`.

---

## [1.0.3] — 2026-04-19

### Added
- **Natural language trigger phrases in `collection.json`.** API entries now include trigger arrays that map conversational phrases to capabilities, powering the routing layer introduced in agent-index-core 3.0.5. Members can say things like "create a strategy" or "run my briefing" instead of using `@ai:` alias syntax. Triggers are customizable per-member via `routing.json`.

---

## [1.0.2] — 2026-03-31

### Changed
- Clarified sharing model description in README: private strategies now explicitly noted as local workspace (`members/{hash}/strategies/`, native file tools), not remote. Aligns with tool-selection guidance already present in task workflows.

---

## [1.0.1] — 2026-03-26

### Changed
- Version bump for infrastructure alignment (see 1.0.0 changelog below).

---

## [1.0.0] — 2026-03-25

Initial release.

### Infrastructure Alignment — Remote Filesystem Migration

Aligned with agent-index v2.0.0 remote filesystem architecture. No functional changes to strategy workflows.

**Breaking:** Requires `agent_index_min_version: 2.0.0`. Remote filesystem access via `aifs_*` tools must be available for shared strategy operations.

- Updated `agent_index_min_version` in `collection.json` from `1.0.0` to `2.0.0`
- Added `reads_from` and `writes_to` paths to `share-strategy.md` (was `null` despite `produces_shared_artifacts: true`)
- Updated `collection-setup.md` prerequisites and parameter descriptions to reference remote filesystem and `aifs_*` MCP tools
- Added tool selection guidance to all task workflows: private workspace operations use native Read/Write tools; shared strategy operations use `aifs_*` MCP tools (`aifs_read`, `aifs_write`, `aifs_exists`)
- Updated `share-strategy.md` workflow steps with explicit `aifs_exists`, `aifs_read`, `aifs_write` references for shared space operations
- Updated manifest operations (`strategies-manifest.json`) across all relevant tasks to specify `aifs_read`/`aifs_write`
- Updated `README.md` sharing model section to reference remote filesystem access via `aifs_*` MCP tools
- Updated `strategy-tutorial.md` to reference "remote filesystem" instead of "filesystem"
