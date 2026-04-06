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
- The remote filesystem is accessible (via `aifs_*` MCP tools)
- The `/shared/strategies/` directory exists and is writable on the remote filesystem (or you have IT create it before proceeding)

---

## Org-Level Parameters

### Shared Strategies Directory

**shared_strategies_path**
- Description: The remote filesystem path where shared strategies are stored (accessed via `aifs_*` MCP tools). When a member shares a strategy for collaboration, it lands here. Private strategies remain in the member's own workspace.
- Applies to: share-strategy, run-briefing (for shared strategies), manage-opportunities (for shared strategies)
- Interview prompt: "Where should shared strategies be stored? The default is `/shared/strategies/` — does that work for your org, or do you need a different path?"
- Accepted values: Any valid remote filesystem path (under `/shared/`)
- Default: `/shared/strategies/`
- Implication of choices: All members with access to this collection can see shared strategies at this path. Changing this after strategies have been shared requires moving files manually.

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
- Description: Controls whether strategies must start as private drafts in the member's workspace, or whether members can create strategies directly in the shared space.
- Applies to: create-strategy, share-strategy
- Interview prompt: "Should strategies always start as private drafts before being shared? Or should members be able to create strategies directly in the shared space?"
  - **Private first** — strategies always start in the member's private workspace. They must be explicitly shared via '@ai:share-strategy' to become visible to collaborators.
  - **Either** — members can choose to start private or create directly in the shared space.
- Accepted values: `private_first` | `either`
- Default: `private_first`
- Implication of choices:
  - `private_first`: Members always begin with private exploration. Sharing is an explicit, intentional step. Best when strategies involve sensitive thinking that should be developed before exposure.
  - `either`: More flexible. Members who are building a strategy explicitly for collaboration can skip the private stage.

---

## Setup Completion

1. Write all collected parameter values to `collection-setup-responses.md`
2. Verify that the `shared_strategies_path` directory exists on the remote filesystem via `aifs_exists`. If it does not exist, prompt the admin: "The directory `{path}` does not exist yet on the remote filesystem. Would you like me to note it as a prerequisite for IT to create, or do you want to use a different path?"
3. Initialize `strategies-manifest.json` at `{shared_strategies_path}/strategies-manifest.json` via `aifs_write` if it does not already exist (check via `aifs_exists`):
```json
{
  "last_updated": "{today}",
  "strategies": []
}
```
4. Confirm to admin with a summary of everything configured:
> "Strategy collection is configured. Here's what was set:
> - Shared strategies location: {shared_strategies_path}
> - Default items per briefing run: {default_items_per_run}
> - Private stage requirement: {private_first/either}
>
> Members can now install the strategy tasks via '@ai:setup'. They'll be able to create strategies privately and share them to {shared_strategies_path} when ready."

---

## Upgrade Behavior

### Preserved Responses
- `shared_strategies_path` — always preserved; changing this would orphan existing shared strategies
- `default_items_per_run` — preserved; safe to change at any time
- `strategies_require_private_stage` — preserved; changing does not affect existing strategies

### Reset on Upgrade
- None — all responses are preserved across upgrades by default

### Requires Admin Attention
- Any new parameters introduced in a new version are surfaced to the admin during upgrade

### Requires Member Attention
None for PATCH/MINOR upgrades. MAJOR version upgrades will document required member actions here.

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new MAJOR versions are published.
