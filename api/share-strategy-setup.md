---
name: share-strategy-setup
type: setup
version: 1.0.2
collection: strategy
description: Setup for the share-strategy task
target: share-strategy
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed and that the member has at least one private strategy to share. No member-specific configuration is required.

---

## Pre-Setup Checks
- Collection setup completed (verify `collection-setup-responses.md` exists)
- At least one private strategy exists in the member's workspace
- Shared strategies path is accessible and writable

---

## Parameters
No member-configurable parameters.

---

## Setup Completion
1. Verify `collection-setup-responses.md` exists in the collection's setup directory
2. Register entry in `member-index.json` with alias `@ai:share-strategy`
3. Confirm to member: "You can now share your strategies for collaboration. Run '@ai:share-strategy' to promote a private strategy to the shared space and invite collaborators."

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
