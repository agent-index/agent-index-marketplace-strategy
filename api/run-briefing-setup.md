---
name: run-briefing-setup
type: setup
version: 1.0.2
collection: strategy
description: Setup for the run-briefing task
target: run-briefing
target_type: task
upgrade_compatible: true
---

## Setup Overview
Validates that the Strategy collection setup has been completed and that the member has at least one strategy with configured information sources. No member-specific configuration is required. Email and web search connectivity are conditional.

---

## Pre-Setup Checks
- Collection setup completed (verify `collection-setup-responses.md` exists)
- At least one strategy exists in the member's workspace
- At least one information source is configured for the strategy

---

## Parameters
No member-configurable parameters.

---

## Setup Completion
1. Verify `collection-setup-responses.md` exists in the collection's setup directory
2. Register entry in `member-index.json` with alias `@ai:run-briefing`
3. Confirm to member: "You can now run briefings to process source materials. Run '@ai:run-briefing' to evaluate incoming items against your strategy and surface new opportunities."

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
