---
name: strategy-review
type: task
version: 1.0.2
collection: strategy
description: Deep review of all active opportunities against the current canonical strategy reference — without pulling new source material. Used for periodic resets, quarterly reviews, or after significant strategy edits.
stateful: false
produces_artifacts: true
produces_shared_artifacts: false
dependencies:
  skills: []
  tasks: []
external_dependencies: []
reads_from: null
writes_to: null
---

## About This Task

A strategy review re-evaluates the full opportunity landscape against the current state of the canonical strategy reference — without ingesting new source material. While briefings are about processing new information, reviews are about stepping back and looking at the big picture.

This is particularly valuable after editing the strategy (a changed pillar may reshape which opportunities matter), at regular intervals (monthly or quarterly), or when the member senses that the opportunity registry has drifted from the current strategic direction.

The review produces a dated document similar to a briefing, but focused on reassessment rather than new discovery.

### Inputs

The member identifies which strategy to review. Optionally, the member can scope the review to specific statuses or priority levels.

### Outputs

- `{strategy-path}/{slug}/briefings/{YYYY-MM-DD}-review-{NNN}.md` — the review document
- Updated `opportunities.json` — reassessed opportunities with updated evaluations
- Updated `state/current-context.md`

### Cadence & Triggers

Manual invocation. Suggested cadence: monthly or quarterly, or after any significant edit to the canonical strategy reference. The system may suggest a review at session start if the strategy has been edited since the last review.

---

## Workflow

### Step 1: Identify Strategy and Load Context

Read `collection-setup-responses.md` via `aifs_read` to get `shared_strategies_path`.

**Tool selection:** Operations on the member's private workspace (`/members/{member_hash}/strategies/`) use native Read/Write tools. Operations on the shared strategies path (`{shared_strategies_path}`) use `aifs_*` tools (e.g., `aifs_read`, `aifs_write`, `aifs_exists`).

If the member named a strategy: find it. If not: list available strategies and ask.

Read (using the appropriate tool based on location):
- `strategy.md` — the canonical reference (load fully)
- `opportunities.json` — the current registry
- `strategy-changelog.jsonl` — to identify recent strategy changes
- `state/current-context.md` — rolling context

If the opportunity registry is empty: "No opportunities to review. Run '@ai:run-briefing' to surface opportunities first." Halt.

**On success:** Proceed to Step 2.

---

### Step 2: Scope the Review

Present a summary:

> **Strategy Review: {name}**
> Total opportunities: {count}
> Active: {N}, Pursued: {N}, Deferred: {N}, Dismissed: {N}, Reassessing: {N}
> {if strategy edited since last review/briefing}: Strategy was updated on {date}: {summary of recent changes from changelog}
>
> Review all active and pursued opportunities, or focus on a subset?

The member can:
- **Review all** — evaluate all non-dismissed opportunities (active, pursued, reassessing, deferred)
- **Focus on active/pursued only** — skip deferred opportunities
- **Focus on specific priorities** — "Review priority 1 and 2 only"
- **Review deferred** — re-evaluate deferred opportunities specifically to see if any should be reactivated

Record the scope. Default: all active and pursued.

**On success:** Proceed to Step 3.

---

### Step 3: Evaluate Opportunities

For each opportunity in scope, evaluate against the current canonical strategy reference:

1. **Alignment check** — does this opportunity still align with the strategy's current vision, pillars, and objectives? If the strategy has been edited since the opportunity was identified, this is especially important.

2. **Priority assessment** — given the current strategic context, is the current priority still appropriate? Suggest an adjustment if warranted.

3. **Status assessment** — should this opportunity's status change? Should a deferred opportunity be reactivated? Should an active opportunity be dismissed?

4. **Staleness check** — how long has it been since this opportunity was evaluated? Flag opportunities that haven't been reviewed in over 30 days.

Group the results:

- **No change needed** — opportunity is well-aligned, priority and status are appropriate
- **Priority adjustment suggested** — opportunity is still valid but priority should shift
- **Status change suggested** — opportunity should move to a different status
- **Strategy misalignment** — opportunity no longer aligns with the current strategy direction

**On success:** Proceed to Step 4.

---

### Step 4: Present Review to Member

Present the review findings:

> **Strategy Review: {name}**
> **Date:** {today}
> **Scope:** {what was reviewed}
> **Opportunities reviewed:** {count}
>
> ## Well-Aligned (No Changes)
> {list of opportunities that need no adjustment — ID, title, priority}
>
> ## Suggested Priority Adjustments
> {for each}:
> **{OPP-XXX}: {title}** — current priority {N}, suggest {M}
> Reason: {why the priority should change}
>
> ## Suggested Status Changes
> {for each}:
> **{OPP-XXX}: {title}** — current: {status}, suggest: {new status}
> Reason: {why}
>
> ## Strategy Misalignment
> {for each}:
> **{OPP-XXX}: {title}**
> {explanation of why this no longer aligns}
> Suggest: dismiss / defer / reframe

**On success:** Proceed to Step 5.

---

### Step 5: Member Decisions

For each suggested change, get the member's decision:

For priority adjustments:
"**{OPP-XXX}: {title}** — adjust priority from {old} to {suggested}?"
- Accept, choose a different priority, or keep as-is.

For status changes:
"**{OPP-XXX}: {title}** — change status from {old} to {suggested}?"
- Accept, choose a different status, or keep as-is.

For misaligned opportunities:
"**{OPP-XXX}: {title}** — this doesn't seem to align anymore. Dismiss, defer, or keep active?"
- The member decides.

Unlike briefing runs, review decisions do not block finalization. The member can acknowledge findings without acting on every suggestion: "Noted, I'll keep it as-is for now." This is acceptable — reviews are about informed awareness, not forced action.

**On success:** Proceed to Step 6.

---

### Step 6: Write Results

1. Write the review document to `{strategy-path}/{slug}/briefings/{YYYY-MM-DD}-review-{NNN}.md`:

   ```
   ---
   review_id: "{YYYY-MM-DD}-review-{NNN}"
   strategy: "{strategy slug}"
   date: "{today YYYY-MM-DD}"
   type: "review"
   author:
     display_name: "{display_name}"
     member_hash: "{member_hash}"
   scope: "{description of scope}"
   opportunities_reviewed: {count}
   changes_made: {count}
   ---

   ## Review Summary

   {narrative summary of the review findings — what's well-aligned, what needed adjustment, what's misaligned}

   ## Changes Made

   {for each opportunity where a change was applied}:
   - **{OPP-XXX}: {title}** — {description of change and reason}

   ## No Changes

   {list of opportunities reviewed but unchanged}

   ## Suggestions Deferred

   {if any suggestions were acknowledged but not acted on}:
   - **{OPP-XXX}: {title}** — {suggestion that was deferred}
   ```

2. Update `opportunities.json`:
   - Apply all confirmed changes (priority, status)
   - For every opportunity in scope: update `last_evaluated_date` to today
   - For changed opportunities: update `last_updated_date` to today
   - Write the file

3. Update `state/current-context.md` with the review findings.

4. Append to `strategy-changelog.jsonl`:
   ```json
   {
     "timestamp": "{ISO 8601}",
     "type": "strategy_review",
     "author_hash": "{member_hash}",
     "author_name": "{display_name}",
     "review_id": "{review_id}",
     "opportunities_reviewed": {count},
     "changes_made": {count},
     "summary": "Strategy review: {count} opportunities reviewed, {count} changes made"
   }
   ```

5. If the strategy is shared: update `strategies-manifest.json` via `aifs_read`/`aifs_write` with current `opportunity_count`.

6. Confirm:
   > "Review complete. {N} opportunities reviewed, {M} changes applied. The review is saved at `{review path}`."

---

## Directives

### Behavior

Reviews should feel reflective, not mechanical. Help the member think about the strategic landscape holistically. "Looking at your full opportunity set, there's a concentration of priority 1 items around the platform ecosystem pillar — that might be exactly right given your strategy, or it might mean other pillars are underserved."

When the strategy has been recently edited, explicitly connect the edits to opportunity alignment: "You recently added 'regulatory compliance' as a constraint. {OPP-007} may be affected because it involves entering a regulated market — worth considering whether the priority should change."

### Constraints

Never pull new source material during a review. Reviews evaluate what's already known. New information comes from briefings.

Never create new opportunities from a review. If the review process suggests a new opportunity, note it in the review document and suggest the member capture it in their next briefing or create it manually.

Never auto-apply changes. Every change requires member acknowledgment.

### Edge Cases

If all opportunities are well-aligned and no changes are needed: say so clearly — "Everything looks well-aligned with your current strategy. No changes suggested." Still write the review document to record that a review was performed.

If the member has never edited the strategy and all opportunities were recently evaluated (last briefing within a week): note that a review may not be necessary yet — "Your last briefing was {N} days ago and the strategy hasn't changed. A review might be more valuable after some time has passed or after a strategy edit. Want to proceed anyway?"

If the review reveals that most opportunities are misaligned: this may indicate the strategy itself has shifted implicitly. Note: "A lot of your opportunities don't seem to align well with the current canonical strategy. This might mean the strategy document needs updating to reflect where your thinking has actually gone. Consider running '@ai:edit-strategy' first."
