---
name: edit-strategy
type: task
version: 1.0.2
collection: strategy
description: Modifies the canonical strategy reference document. Warns non-owners but does not block. Every edit is logged to the append-only changelog with who, what, and why.
stateful: false
produces_artifacts: false
produces_shared_artifacts: false
dependencies:
  skills: []
  tasks: []
external_dependencies: []
reads_from: null
writes_to: null
---

## About This Task

The canonical strategy document is the foundation that all briefings, opportunity evaluations, and strategic thinking are measured against. Editing it is significant — every change shifts the lens through which information is evaluated.

This task allows the member to update any section of the strategy, add new content to previously skipped sections, or revise existing content. Every edit is logged to `strategy-changelog.jsonl` with who made the change, what changed, and why. This creates a full history of how the strategy evolved over time.

Only the strategy owner should edit the canonical reference under normal circumstances. If a collaborator invokes this task on a shared strategy they don't own, the system warns them and suggests going through the owner. It does not block — filesystem permissions are the real enforcement layer.

### Inputs

The member identifies which strategy to edit and provides the changes they want to make.

### Outputs

- Updated `strategy.md`
- New entry in `strategy-changelog.jsonl`
- Updated `state/current-context.md`

### Cadence & Triggers

On demand, whenever the member needs to update their strategic reference.

---

## Workflow

### Step 1: Identify Strategy

Read `collection-setup-responses.md` via `aifs_read` to get `shared_strategies_path`.

**Tool selection:** Operations on the member's private workspace (`/members/{member_hash}/strategies/`) use native Read/Write tools. Operations on the shared strategies path (`{shared_strategies_path}`) use `aifs_*` tools (e.g., `aifs_read`, `aifs_write`, `aifs_exists`).

If the member named a strategy in their invocation: search for it in both the member's private workspace (`/members/{member_hash}/strategies/`) and the shared space (`{shared_strategies_path}`).

If the member did not name a strategy: list all strategies the member has access to (private + shared) and ask which one to edit.

If not found: surface "I couldn't find a strategy matching '{input}'. Say '@ai:create-strategy' to create a new one." Halt.

Read the full `strategy.md` for the identified strategy (using the appropriate tool based on location).

**On success:** Proceed to Step 2.

---

### Step 2: Check Ownership

Extract the `owner.member_hash` from the strategy's frontmatter. Compare to the running member's `member_hash`.

**If the member is the owner:** Proceed to Step 3.

**If the member is not the owner:** Surface: "This strategy is owned by {owner display_name}. Changes to the canonical reference should go through the owner. Do you want to proceed anyway?"
- If yes: proceed to Step 3.
- If no: halt. Suggest: "You can reach out to {owner display_name} with your suggested changes."

---

### Step 3: Determine What to Edit

Present the current state of the strategy — section titles and a brief summary of each section's content (or note that it's not yet defined).

Ask: "What would you like to change?"

The member can:
- **Update an existing section** — provide new or revised content for any section
- **Fill in a skipped section** — provide content for a section that was left as "*Not yet defined.*"
- **Add supplementary content** — if the member describes something that doesn't fit the standard sections, add it as a new section at the end of the document

Accept changes conversationally. The member may describe changes in narrative form ("I want to update the competitive landscape to reflect the new entrant we found last week") or provide direct replacement text.

For each change, confirm what Claude understood: "I'll update the Competitive Landscape section to include {summary of change}. Does that capture it?"

Continue until the member says they're done.

**On success:** Proceed to Step 4.

---

### Step 4: Collect Rationale

Ask: "What's driving these changes? A brief note for the changelog — it helps when looking back at how the strategy evolved."

Accept any non-empty string. If the member declines to provide rationale, record "No rationale provided."

**On success:** Proceed to Step 5.

---

### Step 5: Confirm and Write

Present a summary of all changes:

> **Strategy Edit Summary**
> Strategy: {name}
> Sections changed: {list}
> Rationale: {rationale}
>
> Confirm?

Wait for explicit confirmation.

On confirmation:

1. Update `strategy.md` with all changes. Update `last_updated` in the frontmatter to today.

2. Append to `strategy-changelog.jsonl`:
   ```json
   {
     "timestamp": "{ISO 8601}",
     "type": "strategy_edited",
     "author_hash": "{member_hash}",
     "author_name": "{display_name}",
     "sections_changed": ["{list of sections modified}"],
     "rationale": "{rationale}",
     "summary": "{brief description of what changed}"
   }
   ```

3. Update `state/current-context.md` — regenerate the Strategic Position summary to reflect the updated strategy content.

4. If the strategy is shared: update the `strategies-manifest.json` entry via `aifs_read`/`aifs_write` (if any fields have changed).

5. Confirm to member:
   > "Strategy '{name}' has been updated. {N} section(s) changed. The changelog has been updated."

**On any write failure:** Surface the specific file that failed. Do not leave `strategy.md` and the changelog out of sync — if the strategy file was updated but the changelog write fails, warn the member.

---

## Directives

### Behavior

Strategy edits are significant. Help the member think through the implications. If they're changing a strategic pillar, note: "Changing a pillar may affect how future briefings evaluate opportunities. Existing opportunities tied to the previous framing will still be in the registry — you might want to run '@ai:strategy-review' afterward to re-evaluate them."

Keep the editing flow conversational. The member doesn't need to provide a perfectly formatted replacement — they can describe what they want changed and Claude drafts the new section content for confirmation.

When showing the current state, be concise — don't read the entire document back. Section titles with one-line summaries are sufficient for the member to orient.

### Constraints

Never write changes before the Step 5 confirmation.

Never skip the changelog entry. Every edit must be logged, even if the member declines to provide rationale.

Never silently merge changes. Every proposed change must be confirmed by the member before writing.

### Edge Cases

If the member tries to edit a strategy that has been shared but they're accessing the private copy: warn them that the private copy is a pre-share snapshot and the active version is in the shared space. Offer to edit the shared version instead.

If the strategy.md file is corrupted or unparseable: surface the issue and suggest the member check the file manually or contact their org admin.

If the member wants to make a change that contradicts an existing section (e.g., adding a pillar that conflicts with a stated constraint): note the tension but don't block. "I notice this new pillar may conflict with the constraint about {constraint}. Want to update the constraints section as well, or is the tension intentional?"
