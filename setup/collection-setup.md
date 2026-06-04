---
name: strategy-collection-setup
type: collection-setup
version: 1.0.2
collection: strategy
description: Org-admin setup interview for the Strategy collection
upgrade_compatible: true
---

## Collection Setup Overview

This setup configures how the Strategy collection works across your org. It covers shared strategy storage, default items-per-briefing limits, the opportunity priority scale, and whether strategies must start as private before being shared. This takes about five minutes and can be updated later if your needs change.

---

## Prerequisites

- You have org admin credentials in agent-index
- The remote filesystem is accessible (via `aifs_*` tools)
- Agent-index-core 3.8.0+ and filesystem adapter 2.5.0+ are installed (ID anchors + `member_folder_id` support); `/shared/strategies-index/` will be created during Setup Completion if missing

---

## Org-Level Parameters

### Shared Strategy Model (no path to configure — v1.1.0+)

Shared strategy **content** lives in each owner's private member space (`id:{member_folder_id}/strategies/{slug}/`) and is shared via per-person grants — *not* in a common `/shared` directory. The only org-level location is the **discovery index** at the fixed path `/shared/strategies-index/` (one pointer file per shared strategy), whose `all@` writer grant is declared in this collection's `collaborative-acls.json` and provisioned by `install-collection` Step 5.5. There is nothing to interview the admin about for storage; note this model in the confirmation. (See standards.md § "Addressing: paths vs. ID anchors".)

---

### Default Items Per Briefing Run

**default_items_per_run**
- Description: The default maximum number of source items processed per briefing run. Members can override this per source or per run. This is a sensible default, not a hard limit.
- Applies to: run-briefing
- Interview prompt: "How many source items should a briefing run process by default? This keeps runs manageable. Members can override per source or per run. The default is 10."
- Accepted values: Any positive integer
- Default: `10`
- Implication of choices: Higher numbers produce more comprehensive briefings but take longer. Lower numbers are faster but may miss items that get picked up in subsequent runs.

---

### Strategies Require Private Stage

**strategies_require_private_stage**
- Description: Controls whether strategies must start as private drafts in the member's workspace, or whether members can choose to share immediately at creation (creation is still private; share-strategy runs right after).
- Applies to: create-strategy, share-strategy
- Interview prompt: "Should strategies always start as private drafts before being shared? Or should members be able to share a strategy immediately when they create it?"
  - **Private first** — strategies always start in the member's private workspace. They must be explicitly shared via '@ai:share-strategy' to become visible to anyone else.
  - **Either** — members can choose to start private or share right away (create-strategy hands off to share-strategy immediately).
- Accepted values: `private_first` | `either`
- Default: `private_first`
- Implication of choices:
  - `private_first`: Members always begin with private exploration. Sharing is an explicit, intentional step. Best when strategies involve sensitive thinking that should be developed before exposure.
  - `either`: More flexible. Members who are building a strategy explicitly for collaboration can skip the deliberate-share step. Either way, content lives in the owner's member space and is visible only to people explicitly granted access.

---

## Setup Completion

1. Write all collected parameter values to `collection-setup-responses.md`
2. Ensure the discovery index folder `/shared/strategies-index/` exists: check via `aifs_exists`; if missing, create it by writing a placeholder `aifs_write '{"path":"/shared/strategies-index/README.md","content":"Discovery index for shared strategies. One pointer file per shared strategy — content lives in the owner's member space. Managed by the strategy collection; do not place strategy content here."}'`. This must exist before `install-collection` Step 5.5 applies the `collaborative-acls.json` grant (`all@` writer on this folder).
3. Confirm to admin with a summary of everything configured:
> "Strategy collection is configured. Here's what was set:
> - Default items per briefing run: {default_items_per_run}
> - Private stage requirement: {private_first/either}
>
> Strategy content always lives in each owner's private member space; sharing applies per-person grants and writes a discovery pointer to `/shared/strategies-index/`. Members can now install the strategy tasks via '@ai:setup'."

---

## Upgrade Behavior

### Preserved Responses
- `default_items_per_run` — preserved; safe to change at any time
- `strategies_require_private_stage` — preserved; changing does not affect existing strategies

### Reset on Upgrade
- `shared_strategies_path` — **removed in v1.1.0**. The shared-directory model is retired; delete this response from `collection-setup-responses.md` during upgrade.

### Requires Admin Attention
- **v1.0.x → v1.1.0**: Run Setup Completion step 2 (ensure `/shared/strategies-index/` exists), then re-run `install-collection` (or its Step 5.5) so the `collaborative-acls.json` grant is applied. If any strategies were already shared under the old `/shared/strategies/` path, the owner should re-share them via '@ai:share-strategy' (which moves content to their member space and writes pointers); the old directory can then be archived.
- Any new parameters introduced in a new version are surfaced to the admin during upgrade

### Requires Member Attention
- **v1.0.x → v1.1.0**: Members who previously shared strategies should re-share them via '@ai:share-strategy' under the new owned-content model. Private strategies are unaffected.

### Migration Notes
- v1.0.x → v1.1.0: shared strategy storage moved from a common `/shared/strategies/` directory (with `strategies-manifest.json`) to owner member spaces with per-person grants and a pointer index at `/shared/strategies-index/`. No automatic data migration is performed; re-sharing handles relocation per strategy.
