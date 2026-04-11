---
name: strategy-review-setup
type: setup
version: 1.0.2
collection: strategy
description: Setup for the strategy-review task
target: strategy-review
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed and that the member has at least one strategy with active opportunities. No member-specific configuration is required.

---

## Pre-Setup Checks
- Collection setup completed (verify `collection-setup-responses.md` exists)
- At least one strategy exists in the member's workspace
- At least one opportunity exists in the strategy's registry

---

## Parameters
No member-configurable parameters.

---

## Setup Completion
1. Verify `collection-setup-responses.md` exists in the collection's setup directory
2. Register entry in `member-index.json` with alias `@ai:strategy-review`
3. Confirm to member: "You can now conduct deep reviews of your opportunities. Run '@ai:strategy-review' to evaluate all active opportunities against your canonical strategy reference."

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
