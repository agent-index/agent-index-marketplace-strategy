---
name: share-strategy
type: task
version: 1.3.0
collection: strategy
description: Shares a private strategy from the member's own remote space using per-person grants — share with X (read), make X a collaborator (read+write), or share with the org (everyone reads). The owner retains ownership; content never moves to /shared. Discovery happens via a pointer file in /shared/strategies-index/.
stateful: false
produces_artifacts: false
produces_shared_artifacts: true
dependencies:
  skills: []
  tasks: []
external_dependencies: []
reads_from: "id:{member_folder_id}/strategies/, /shared/strategies-index/"
writes_to: "id:{member_folder_id}/strategies/, /shared/strategies-index/"
---

## About This Task

Sharing a strategy makes it visible to specific people, or to the whole org — **without moving it out of your control**. The strategy is copied from your local private workspace into **your own private remote member space** (`id:{member_folder_id}/strategies/{slug}/`) — your `Agent-Index-Private` folder in **your own My Drive**, which nobody else can see by default. You are the Drive owner of that folder, which is exactly what makes the grants below possible (core 3.9.0 model; Shared-Drive folders can only be shared by drive Managers). Sharing is then purely additive grants on that one folder:

- **Share with X** → X can **read** it.
- **Make X a collaborator** → X can **read and write** (run briefings, manage sources, evaluate opportunities).
- **Share with the org** → everyone can **read** it.

Only you (the owner) and your named collaborators can ever write. The org-wide level is read-only. Editing the canonical `strategy.md` remains an owner action — collaborators are warned and redirected to you (a task-level convention, as before).

Discovery: each shared strategy gets one **pointer file** at `/shared/strategies-index/{owner_hash}-{slug}.json` so others can find it and open it by its Drive folder ID. Note the pointer (title, owner, scope) is org-readable — the *existence* of a shared strategy is visible org-wide even when its *content* is restricted to named people.

The original local private copy is retained as a snapshot with a cross-reference.

### Inputs

The member identifies which private strategy to share, the sharing level(s), and the people (for per-person levels).

### Outputs

- Strategy copied to `id:{member_folder_id}/strategies/{slug}/` (the owner's private remote space)
- Per-person / org grants applied through the core permission-changes procedure (owner confirms the exact changes; each verified)
- Pointer written to `/shared/strategies-index/{owner_hash}-{slug}.json`
- Local private copy updated with cross-reference

---

## Workflow

### Step 1: Resolve Identity and Strategy

Read `member-index.json` (local) for `member_hash` and **`member_folder_id`**. If `member_folder_id` is missing: run `@ai:update` (its Migration 2 creates your `Agent-Index-Private` My Drive space and caches the ID) or `@ai:member-bootstrap` — then retry. There is no admin backfill under the 3.9.0 model; the member self-provisions. (See `agent-index-core/runtime-reference.md` § "Addressing: Owned Content and Sharing".)

**Tool selection:** the local private workspace (`members/{member_hash}/strategies/`) uses native Read/Write. The member's remote space (`id:{member_folder_id}/...`) and the pointer index (`/shared/strategies-index/`) use `aifs_*`.

If the member named a strategy: find it in their local private workspace. If not: list their private strategies and ask which to share. If already shared (private copy has `shared: true` + `shared_path`): "That strategy is already shared. Use '@ai:edit-strategy' to change who has access." Halt.

### Step 2: Review Before Sharing

Present the strategy summary (name, vision, sources, opportunities, briefings) and ask whether it's ready. If the member wants changes first, direct them to '@ai:edit-strategy' and halt. Unchanged from prior versions.

### Step 3: Choose Sharing Level(s) and People

Ask: "How do you want to share it?"

- **Share with specific people (read-only):** collect names.

  **Recipients.**
  - Resolve each against the members registry (case-insensitive partial match on `display_name`, confirm) → their `sharing_identity`.
  - A name that doesn't resolve → ask for their email address. Never guess.
  - An address outside the registry is allowed. The permission-changes table marks it **outside your org**, and it's the owner's call, as it would be in Drive directly. Tell the owner the recipient opens it from their storage app's "Shared with me", because the strategies index is for org members.
- **Add collaborators (read + write):** same resolution. Collaborators can run briefings, manage sources, and evaluate opportunities in the shared copy.
- **Share with the org (everyone reads):** no names needed.

Levels combine (e.g., org-read + two collaborators). Summarize: who gets read, who gets write, whether the org can read — and remind: "the strategy itself stays in your private space; people only get exactly these grants. Its title/owner will be listed in the org-visible index."

### Step 4: Copy, Grant, Publish Pointer

On confirmation:

1. **Copy, or resume an earlier attempt.** Check `aifs_exists("id:{member_folder_id}/strategies/{slug}")` and `aifs_exists("/shared/strategies-index/{member_hash}-{slug}.json")`:
   - **Folder absent:** copy the entire strategy directory from the local private workspace to `id:{member_folder_id}/strategies/{slug}/` via `aifs_write` (all files: strategy.md, sources.json, opportunities.json, strategy-changelog.jsonl, briefings/, state/). Write the copied `strategy.md` as-is: no `shared: true` and no `collaborators` yet (those are written in step 3, after the gate).
   - **Folder present, no pointer for this owner + slug:** this is an earlier share of this strategy that stopped before any grant was satisfied (declined, cancelled or failed). **Resume it** instead of treating it as a collision: tell the member "Picking up your earlier share of '{name}'." Re-copy any file whose local content differs from the remote copy (the local private copy is the source of truth until sharing completes), then continue at step 2, which re-stats the folder and reuses its `folder_id` / `item_drive_id`.
   - **Folder present and a pointer exists:** the slug is already shared. If it is this strategy, say "That strategy is already shared. Use '@ai:edit-strategy' to change who has access." Otherwise ask the member to choose a different slug. Halt.
2. **Capture the folder ID and home drive:** `aifs_stat("id:{member_folder_id}/strategies/{slug}")` → record its `id` as `folder_id` (adapter 2.5.0+) **and the returned `drive_id` as `item_drive_id` (adapter 2.3.0+)**. The shared copy lives in the owner's own My Drive, so capturing `drive_id` is what lets a recipient open it cross-drive (C.1.3 `crossdriveread`). A bare `id:{folder_id}` resolves against the *recipient's* drive and 404s on OneDrive even with the grant present. On a resume, these are the existing folder's values.
3. **Apply the grants.** Follow `/agent-index-core/internal/permission-changes.md`:
   - `calling_task: "share-strategy"`, `confirmer: "owner"`
   - one `share` per grant on resource `id:{folder_id}`: the **exact** strategy-folder Drive ID captured in step 2, not the anchor-plus-relative-path form. Granting the precise folder is the least-privilege surface.
   - `resource_label` "Strategy '{name}'", `is_folder: true`, `source: "member_request"`:
     - each read-person → `role: "reader"`
     - each collaborator → `role: "writer"`
     - org level → recipient `{all_members_group}`, `role: "reader"`

   **Gate:** a grant counts only if its row is `applied` or `skipped_noop`. Never proceed on "pending". Nothing that records access (the `shared` flag, `collaborators`, the pointer, the changelog event, the local cross-reference, the confirmation) is written before the procedure returns, and each lists only satisfied rows. A pointer written before the grants exist advertises access that doesn't exist; recipients would discover the strategy and hit ACCESS_DENIED.
   - **No row satisfied** (no row is `applied` or `skipped_noop` — judge by row states, not `run_result`, since `none` can still include `skipped_noop` rows; declined, cancelled or failed; a cancelled table declines every row): nothing was granted. **Leave the remote copy exactly as it is** (don't mark it, don't delete it), write NO pointer, and don't touch the local private copy. Tell the member: "'{name}' is not shared. Nobody has been given access. Run '@ai:share-strategy' again when you're ready; it picks up where this left off." The copied folder sits in the member's own My Drive, visible to nobody else; they can delete it there if they'd rather not share after all.
   - **Some rows satisfied** (`partial`): continue with ONLY those. Report the rest with the procedure's summary lines and offer to add them later via '@ai:edit-strategy'.
3a. **Mark the shared copy:** update the shared copy's `strategy.md`: `shared: true`, `shared_path: null`, `last_updated` today, and `collaborators` (display_name / member_hash or null / email / level / added_date) for satisfied rows only.
4. **Write the pointer** (`scope` lists only satisfied rows) to `/shared/strategies-index/{owner_hash}-{slug}.json` via `aifs_write`:
   ```json
   {
     "type": "strategy",
     "owner": "{display_name}",
     "owner_hash": "{member_hash}",
     "slug": "{slug}",
     "folder_id": "{folder_id from step 2}",
     "item_drive_id": "{item_drive_id from step 2}",
     "scope": {"org_read": true|false, "readers": ["email", ...], "collaborators": ["email", ...]},
     "title": "{strategy name}",
     "shared_date": "{today}"
   }
   ```
   `item_drive_id` (the owner's home drive for that folder; C.1.3 `crossdriveread`) is what lets a recipient on OneDrive open content that physically lives on the owner's personal drive. Recipients discover shared strategies by reading this index and open them via the cross-drive anchor `id:{item_drive_id}:{folder_id}/...` when `item_drive_id` is present, falling back to the bare `id:{folder_id}/...` only for older pointers that predate it. On gdrive the bare anchor already reaches shared items, so the qualified form is OneDrive parity and harmless there.
5. **Update the local private copy:** `shared: true`, `shared_path: "id:{member_folder_id}/strategies/{slug}/"`, `last_updated` today.
6. **Append** a `strategy_shared` event to the shared copy's `strategy-changelog.jsonl`, naming only the satisfied grantees (readers, collaborators, org-read).
7. Confirm to the member, naming only satisfied grants: who can read, who can write, whether the org can read, and that '@ai:edit-strategy' manages access from here. For any recipient outside the org, add: "They'll find it in their storage app's 'Shared with me'. The strategies index is for org members."

### Un-sharing (reference; performed via edit-strategy)

Revoking access = an `unshare` change through the core permission-changes procedure (owner confirms each removal), then re-read `aifs_get_permissions(id:{folder_id})` and **overwrite** the pointer from what is actually in effect: `scope` lists whoever still has access, and `"scope": "revoked"` is set only when no explicit grants other than the owner remain. Never derive the new scope from the removals you intended or from declined rows. **Never delete the pointer.** It lives on the Shared Drive where members cannot trash (soft-delete convention, `agent-index-core/runtime-reference.md` § "Addressing: Owned Content and Sharing"). The strategy content in your own My Drive is yours to delete, but prefer archive-marking so changelogs and cross-references stay resolvable.

---

## Directives

### Behavior

Sharing exposes the member's strategic thinking — keep the tone encouraging. Be explicit about the three levels and that write access is only ever named collaborators + owner.

### Constraints

- Never share a strategy that is already shared (direct to edit-strategy).
- Never modify the local private copy's content during sharing — only status fields and cross-reference.
- Never call `aifs_share`/`aifs_unshare`/`aifs_transfer_ownership` outside the core permission-changes procedure. The owner confirms the exact changes. Pointers never claim unverified state.
- Never delete pointer files or anything on the Shared Drive (members cannot trash there) — removal semantics on org surfaces are overwrite/mark (soft-delete). Content in the owner's My Drive is owner-deletable but archive-marking is preferred.
- The pointer's existence/title/owner are org-visible by design; if the member objects to even the title being visible, offer to use the slug as the title.

### Edge Cases

- `member_folder_id` missing → direct the member to `@ai:update` / `@ai:member-bootstrap` (self-provisioning; Step 1).
- Collaborator outside the org → allowed by email, flagged in the table. If the backend rejects the address (e.g. the Workspace blocks external sharing), the row is `failed` with the reason; relay it.
- Not every row satisfied → see the step 3 gate: `none` → remote copy left untouched, no pointer, member told it isn't shared and to re-run (the re-run resumes); `partial` → pointer with only the satisfied grants, the rest offered via edit-strategy.
- Slug folder already in the member's own space with no pointer → an earlier unfinished share; resume it (step 1). With a pointer → already shared or a real collision; edit-strategy or a different slug.
- Ownership transfer during sharing → not supported (the folder lives in the owner's member space); transfer = the new owner shares their own copy. Direct the member accordingly.
