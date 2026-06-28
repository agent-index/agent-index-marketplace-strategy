---
name: manage-opportunities-setup
type: setup
version: 1.0.2
collection: strategy
description: Setup for the manage-opportunities task
target: manage-opportunities
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed and that the member has at least one strategy with an opportunities registry. No member-specific configuration is required.

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
2. Register entry in `member-index.json` with alias `@ai:manage-opportunities`
3. Confirm to member: "You can now manage opportunities across your strategies. Run '@ai:manage-opportunities' to view, filter, sort, update, and track opportunities."

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
