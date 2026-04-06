---
name: create-strategy-setup
type: setup
version: 1.0.0
collection: strategy
description: Setup for the create-strategy task
target: create-strategy
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed at the org level, confirming that shared strategies path and strategy creation policies are configured. No member-specific configuration is required.

---

## Pre-Setup Checks
- Collection setup completed (verify `collection-setup-responses.md` exists)
- Shared strategies path is accessible
- Member has write access to private workspace

---

## Parameters
No member-configurable parameters.

---

## Setup Completion
1. Verify `collection-setup-responses.md` exists in the collection's setup directory
2. Register entry in `member-index.json` with alias `@ai:create-strategy`
3. Confirm to member: "You can now create strategies in your org. Run '@ai:create-strategy' to get started."

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
