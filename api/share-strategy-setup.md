---
name: share-strategy-setup
type: setup
version: 1.1.0
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
- `member_folder_id` present in `member-index.json` (if missing, '@ai:member-bootstrap' refreshes it from the registry)
- The pointer index `/shared/strategies-index/` exists (created by collection setup; `all@` writer grant applied at install)

---

## Parameters
No member-configurable parameters.

---

## Setup Completion
1. Verify `collection-setup-responses.md` exists in the collection's setup directory
2. Register entry in `member-index.json` with alias `@ai:share-strategy`
3. Confirm to member: "You can now share your strategies. Run '@ai:share-strategy' to share a private strategy from your own member space — with specific people (read), collaborators (read + write), or the whole org (read)."

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
