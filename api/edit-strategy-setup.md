---
name: edit-strategy-setup
type: setup
version: 1.0.2
collection: strategy
description: Setup for the edit-strategy task
target: edit-strategy
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed and that the member has at least one strategy to edit. No member-specific configuration is required.

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
2. Register entry in `member-index.json` with alias `@ai:edit-strategy`
3. Confirm to member: "You can now edit strategy reference documents. Run '@ai:edit-strategy' to modify any strategy's vision, pillars, objectives, and other sections."

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
