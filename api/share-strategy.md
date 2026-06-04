---
name: share-strategy
type: task
version: 1.1.1
collection: strategy
description: Shares a private strategy from the member's own remote space using per-person grants — share with X (read), make X a collaborator (read+write), or share with the org (everyone reads). The owner retains ownership; content never moves to /shared. Discovery happens via a pointer file in /shared/strategies-index/.
stateful: false
produces_artifacts: false
produces_shared_artifacts: true
dependencies:
  skills: ["permission-change-helper"]
  tasks: []
external_dependencies: []
reads_from: "id:{member_folder_id}/strategies/, /shared/strategies-index/"
writes_to: "id:{member_folder_id}/strategies/, /shared/strategies-index/"
---

## About This Task

Sharing a strategy makes it visible to specific people, or to the whole org — **without moving it out of your control**. The strategy is copied from your local private workspace into **your own private remote member space** (`id:{member_folder_id}/strategies/{slug}/`), which nobody else can see by default. Sharing is then purely additive grants on that one folder:

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
- Per-person / org grants applied via `permission-change-helper` (owner reviews + Accepts)
- Pointer written to `/shared/strategies-index/{owner_hash}-{slug}.json`
- Local private copy updated with cross-reference

---

## Workflow

### Step 1: Resolve Identity and Strategy

Read `member-index.json` (local) for `member_hash` and **`member_folder_id`**. If `member_folder_id` is missing: re-read it from the member's own entry in `/members-registry.json` (known-path read) and cache it. If the registry entry also lacks it, halt: "Your private remote folder isn't registered for ID-anchored access yet — ask your admin to run the member-folder-id backfill." (See standards.md § "Addressing: paths vs. ID anchors".)

**Tool selection:** the local private workspace (`members/{member_hash}/strategies/`) uses native Read/Write. The member's remote space (`id:{member_folder_id}/...`) and the pointer index (`/shared/strategies-index/`) use `aifs_*`.

If the member named a strategy: find it in their local private workspace. If not: list their private strategies and ask which to share. If already shared (private copy has `shared: true` + `shared_path`): "That strategy is already shared. Use '@ai:edit-strategy' to change who has access." Halt.

### Step 2: Review Before Sharing

Present the strategy summary (name, vision, sources, opportunities, briefings) and ask whether it's ready. If the member wants changes first, direct them to '@ai:edit-strategy' and halt. Unchanged from prior versions.

### Step 3: Choose Sharing Level(s) and People

Ask: "How do you want to share it?"

- **Share with specific people (read-only):** collect names; resolve each against the members registry (case-insensitive partial match on `display_name`, confirm; unregistered people cannot be granted — Drive grants need a real account).
- **Add collaborators (read + write):** same resolution. Collaborators can run briefings, manage sources, and evaluate opportunities in the shared copy.
- **Share with the org (everyone reads):** no names needed.

Levels combine (e.g., org-read + two collaborators). Summarize: who gets read, who gets write, whether the org can read — and remind: "the strategy itself stays in your private space; people only get exactly these grants. Its title/owner will be listed in the org-visible index."

### Step 4: Copy, Grant, Publish Pointer

On confirmation:

1. **Copy** the entire strategy directory from the local private workspace to `id:{member_folder_id}/strategies/{slug}/` via `aifs_write` (all files — strategy.md, sources.json, opportunities.json, strategy-changelog.jsonl, briefings/, state/). If `aifs_exists("id:{member_folder_id}/strategies/{slug}")` already: ask to choose a different slug. Halt on collision.
2. **Capture the folder ID:** `aifs_stat("id:{member_folder_id}/strategies/{slug}")` → record its `id` as `folder_id` (adapter 2.5.0+).
3. **Update the shared copy's `strategy.md`:** `shared: true`, `shared_path: null`, populate `collaborators` (display_name/member_hash/email/added_date per person, with their level), `last_updated` today.
4. **Apply the grants** — compose ONE `permission-change-helper` spec with an `op: "share"` per grant on resource `id:{folder_id}` (the **exact** strategy-folder Drive ID captured in step 2 — NOT the anchor-plus-relative-path form; permission-helper-go 0.4.0+ accepts only the bare `id:{folderId}` form, and granting the precise folder is the least-privilege surface):
   - each read-person → `role: "reader"`
   - each collaborator → `role: "writer"`
   - org level → recipient `{all_members_group}`, `role: "reader"`
   The member (owner) reviews the page and **Accepts** — grants apply under their own credentials. Never call `aifs_share` directly.
   **HARD GATE — wait for the outcome file and read it.** Do NOT proceed to step 5 (or any later step) until the helper's outcome JSON reports `"outcome": "applied"`. Never assume success, never proceed "pending Accept" — a pointer written before the grants exist advertises access that does not exist (recipients will discover the strategy and hit ACCESS_DENIED). On `rejected`/`page_closed`/`timed_out`/`validation_error`/any other outcome: nothing was granted — members cannot delete the copied folder; overwrite the copied `strategy.md` with `status: "abandoned-share"`, write NO pointer, and tell the member to re-run when ready (the admin can remove the folder if desired).
5. **Write the pointer** (only after step 4 reports `applied`) to `/shared/strategies-index/{owner_hash}-{slug}.json` via `aifs_write`:
   ```json
   {
     "type": "strategy",
     "owner": "{display_name}",
     "owner_hash": "{member_hash}",
     "slug": "{slug}",
     "folder_id": "{folder_id from step 2}",
     "scope": {"org_read": true|false, "readers": ["email", ...], "collaborators": ["email", ...]},
     "title": "{strategy name}",
     "shared_date": "{today}"
   }
   ```
   Recipients discover shared strategies by reading this index and open them via `id:{folder_id}/...`.
6. **Update the local private copy:** `shared: true`, `shared_path: "id:{member_folder_id}/strategies/{slug}/"`, `last_updated` today.
7. **Append** a `strategy_shared` event to the shared copy's `strategy-changelog.jsonl`.
8. Confirm to the member: who can read, who can write, whether the org can read, and that '@ai:edit-strategy' manages access from here.

### Un-sharing (reference; performed via edit-strategy)

Revoking access = `permission-change-helper` `op: "unshare"` for the person (owner Accepts) + **overwrite** the pointer (update `scope`, or set `"scope": "revoked"` to unshare entirely). **Never delete remote files** — members cannot trash on the Shared Drive (soft-delete convention, standards.md).

---

## Directives

### Behavior

Sharing exposes the member's strategic thinking — keep the tone encouraging. Be explicit about the three levels and that write access is only ever named collaborators + owner.

### Constraints

- Never share a strategy that is already shared (direct to edit-strategy).
- Never modify the local private copy's content during sharing — only status fields and cross-reference.
- Never call `aifs_share`/`aifs_unshare` directly — all grants via `permission-change-helper` with the owner's Accept.
- Never delete remote files (members cannot trash) — all removal semantics are overwrite/mark (soft-delete).
- The pointer's existence/title/owner are org-visible by design; if the member objects to even the title being visible, offer to use the slug as the title.

### Edge Cases

- `member_folder_id` missing from registry → halt with the backfill message (Step 1).
- Unregistered collaborator → cannot be granted (Drive needs a real account); offer to share once they're invited to the org.
- Helper outcome anything other than `applied` (`rejected` / `page_closed` / `timed_out` / `validation_error` / `partial_failure`) → no (or incomplete) grants; see the step 4 hard gate. On `partial_failure`: report which grants applied, write the pointer with ONLY the applied grants in `scope`, and offer to retry the failed ones via edit-strategy.
- Requires `permission-helper-go` **0.4.0+** (ID-anchor resource support). On a `validation_error` mentioning `id:` resources, the member's binary is outdated — direct them to `@ai:update` (binary sync runs in Phase 1).
- Slug collision in the member's own space → choose a different slug.
- Ownership transfer during sharing → not supported in 1.1.0 (the folder lives in the owner's member space); transfer = the new owner shares their own copy. Direct the member accordingly.
