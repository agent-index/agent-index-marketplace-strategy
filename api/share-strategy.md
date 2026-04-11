---
name: share-strategy
type: task
version: 1.0.2
collection: strategy
description: Promotes a private strategy to the shared space and invites collaborators. The original owner retains ownership. Collaborators can do everything except edit the canonical reference without a warning.
stateful: false
produces_artifacts: false
produces_shared_artifacts: true
dependencies:
  skills: []
  tasks: []
external_dependencies: []
reads_from: /shared/strategies/
writes_to: /shared/strategies/
---

## About This Task

Sharing a strategy moves it from the member's private workspace to the org's shared strategy space, making it visible to collaborators. This is an intentional, explicit act — the member decides when their strategic thinking is ready for collaboration.

The sharing model is simple: one owner, everyone else is a collaborator. The owner controls the canonical strategy reference. Collaborators can run briefings, manage sources, evaluate opportunities, and do everything else — but if they try to edit the canonical reference, the system warns them and suggests going through the owner.

The original private copy is retained as a snapshot with a cross-reference to the shared version.

### Inputs

The member identifies which private strategy to share and optionally provides initial collaborators.

### Outputs

- Strategy directory copied to `{shared_strategies_path}/{strategy-slug}/`
- Updated `strategies-manifest.json`
- Private copy updated with `shared_path` cross-reference
- Collaborators recorded in the shared strategy's `strategy.md`

### Cadence & Triggers

On demand, when a member decides their private strategy is ready for collaboration.

---

## Workflow

### Step 1: Read Configuration and Identify Strategy

Read `collection-setup-responses.md` via `aifs_read` to get `shared_strategies_path`.

**Tool selection:** Reading/writing the member's private workspace (`/members/{member_hash}/strategies/`) uses native Read/Write tools. Reading/writing the shared strategies path (`{shared_strategies_path}`) uses `aifs_*` MCP tools (e.g., `aifs_read`, `aifs_write`, `aifs_exists`).

If the member named a strategy: find it in their private workspace (`/members/{member_hash}/strategies/`).

If not: list their private strategies and ask which one to share.

If the strategy is not found in private workspace: check if it's already shared — "That strategy is already in the shared space. You can manage collaborators with '@ai:edit-strategy'." Halt.

Read the full strategy directory contents: `strategy.md`, `sources.json`, `opportunities.json`, `strategy-changelog.jsonl`, all briefings, `state/`.

**On success:** Proceed to Step 2.

---

### Step 2: Review Before Sharing

Present the strategy summary:

> **Share this strategy?**
> Name: {name}
> Vision: {first sentence or two of vision, or "not yet defined"}
> Sources: {count} configured
> Opportunities: {count} tracked ({active count} active)
> Briefings: {count} completed
>
> This will make it visible at `{shared_strategies_path}/{slug}/`.

Ask: "Would you like to make any changes before sharing, or is it ready?"

If the member wants changes: direct them to '@ai:edit-strategy' first, then come back to share. Do not perform edits within this task.

**On success:** Proceed to Step 3.

---

### Step 3: Invite Collaborators

Ask: "Would you like to invite collaborators? Collaborators can run briefings, manage sources, and evaluate opportunities. You'll remain the owner — changes to the canonical strategy reference go through you."

If yes, collect collaborators one at a time:
- Ask: "Who would you like to invite?"
- Resolve against the members registry (same pattern as projects: case-insensitive partial match on `display_name`, confirm, handle unregistered).
- Continue until the member says they're done.

If no: proceed with no collaborators. The strategy is shared but not actively assigned to anyone yet.

**On success:** Proceed to Step 4.

---

### Step 4: Confirm and Write

Present a final summary:

> **Sharing '{name}'**
> Location: {shared_strategies_path}/{slug}/
> Owner: {member display_name} (you)
> {if collaborators}: Collaborators: {list of names}
>
> Confirm?

Wait for confirmation.

On confirmation:

1. Check via `aifs_exists` that `{shared_strategies_path}/{slug}/` does not already exist. If it does: "A shared strategy with the slug '{slug}' already exists. Choose a different name or contact your org admin." Halt.

2. Copy the entire strategy directory from the private workspace to `{shared_strategies_path}/{slug}/` via `aifs_write`. All files — strategy.md, sources.json, opportunities.json, strategy-changelog.jsonl, briefings/, state/.

3. Update the shared `strategy.md`:
   - Set `shared: true`
   - Set `shared_path: null` (it IS the shared version)
   - Populate `collaborators` array:
     ```yaml
     collaborators:
       - display_name: "{name}"
         member_hash: "{hash or null}"
         email: "{email or null}"
         added_date: "{today}"
     ```
   - Set `last_updated` to today

4. Update the private `strategy.md`:
   - Set `shared: true`
   - Set `shared_path: "{shared_strategies_path}/{slug}/"` for cross-reference
   - Set `last_updated` to today

5. Read `strategies-manifest.json` at `{shared_strategies_path}` via `aifs_read` and add entry:
   ```json
   {
     "slug": "{slug}",
     "name": "{name}",
     "owner": "{owner display_name}",
     "owner_hash": "{owner member_hash}",
     "created": "{original created date}",
     "shared_date": "{today}",
     "last_briefing": "{most recent briefing date or null}",
     "opportunity_count": {count},
     "collaborators": {count}
   }
   ```
   Update `last_updated` on the manifest. Write the file via `aifs_write`.

6. Append to the shared strategy's `strategy-changelog.jsonl` via `aifs_write`:
   ```json
   {
     "timestamp": "{ISO 8601}",
     "type": "strategy_shared",
     "author_hash": "{member_hash}",
     "author_name": "{display_name}",
     "collaborators_added": ["{list of collaborator names}"],
     "summary": "Strategy shared to {shared_strategies_path}/{slug}/ with {N} collaborators"
   }
   ```

7. Confirm to member:
   > "Strategy '{name}' is now shared. Collaborators can find it at the shared strategies location."
   > {if collaborators}: "{N} collaborators have been added."
   > "You remain the owner. Collaborators can run briefings and manage opportunities. Changes to the canonical reference will go through you."
   > "Manage collaborators anytime with '@ai:edit-strategy'."

---

## Directives

### Behavior

Sharing a strategy is significant — the member is exposing their strategic thinking. Keep the tone encouraging and the process smooth.

When adding collaborators, don't ask for assignments or roles. The sharing model is simple: owner and collaborators. No need for the complexity of role-based access — the ownership warning on canonical reference edits is sufficient governance.

### Constraints

Never share a strategy that is already shared. If the member's private copy has `shared: true` and `shared_path` set, direct them to the shared version.

Never modify the private copy's content during sharing — only update status fields and cross-reference.

Never create a shared strategy that conflicts with an existing slug in the shared space.

### Edge Cases

If `strategies-manifest.json` doesn't exist: halt with "The strategies manifest is missing. Ask your org admin to run the Strategy collection setup."

If a collaborator is unregistered (member_hash is null): record them by name only. Note: "{name} isn't in the org registry yet. They'll be listed as a collaborator but won't be automatically discoverable until they join agent-index."

If the member wants to share but `strategies_require_private_stage` is `either` and they created it directly in shared space originally: this situation shouldn't arise (it would already be shared), but if it does, explain that the strategy is already shared.

If the strategy slug collides with an existing shared strategy: offer to use a different slug or ask the member to rename.

If the member wants to transfer ownership during sharing: accept it. Ask who the new owner should be, resolve against registry, and set them as owner in the shared copy. The original member becomes a collaborator.
