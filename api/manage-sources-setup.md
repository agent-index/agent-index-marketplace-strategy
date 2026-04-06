---
name: manage-sources-setup
type: setup
version: 1.0.0
collection: strategy
description: Setup for the manage-sources task
target: manage-sources
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed and that the member has at least one strategy. No member-specific configuration is required. Email and web search sources are optional and conditionally connected during task execution.

---

## Pre-Setup Checks
- Collection setup completed (verify `collection-setup-responses.md` exists)
- At least one strategy exists in the member's workspace

---

## Parameters
No member-configurable parameters.

---

## Setup Completion
1. Verify `collection-setup-responses.md` exists in the collection's setup directory
2. Register entry in `member-index.json` with alias `@ai:manage-sources`
3. Confirm to member: "You can now add and manage information sources for your strategies. Run '@ai:manage-sources' to configure email, web search, RSS, and other sources."

---

## Upgrade Behavior

### Preserved Responses
N/A.

### Reset on Upgrade
N/A.

### Requires Member Attention
None.

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new versions are published.
