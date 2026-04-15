---
name: manage-opportunities
type: task
version: 1.0.2
collection: strategy
description: View, filter, sort, update, and manage the master opportunity registry for a strategy. Supports priority changes, status transitions, notes, and filtering by priority, status, or date ranges.
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

The opportunity registry is the long-term memory of the strategy system. Every opportunity surfaced by briefings lives here — with its priority, status, dates, and history. This task provides the member with full visibility into and control over the registry.

This is the master list. It can be filtered by priority (1–5), status (pending_review, active, pursued, deferred, dismissed, reassessing), and any of the three dates (identified, last evaluated, last updated). Members use this to track what's on the radar, what's being pursued, what's been set aside, and what needs attention.

### Inputs

The member identifies which strategy to manage opportunities for and what they want to do — view, filter, update, or change status/priority.

### Outputs

Updated `opportunities.json` for the identified strategy.

### Cadence & Triggers

On demand, whenever the member wants to review or manage their opportunity landscape.

---

## Workflow

### Step 1: Identify Strategy

Read `collection-setup-responses.md` via `aifs_read` to get `shared_strategies_path`.

**Tool selection:** Operations on the member's private workspace (`/members/{member_hash}/strategies/`) use native Read/Write tools. Operations on the shared strategies path (`{shared_strategies_path}`) use `aifs_*` tools (e.g., `aifs_read`, `aifs_write`, `aifs_exists`).

If the member named a strategy: find it. If not: list available strategies and ask.

Read `opportunities.json` for the identified strategy (using the appropriate tool based on location).

If the registry is empty: "No opportunities have been surfaced yet. Run '@ai:run-briefing' to evaluate source material and surface opportunities." Halt.

**On success:** Proceed to Step 2.

---

### Step 2: Determine Action

If the member stated what they want to do: proceed to the appropriate action.

If unclear, present a summary dashboard and ask:

> **Opportunity Registry: {strategy name}**
> Total: {count}
> By status: {active: N, pursued: N, pending_review: N, deferred: N, dismissed: N, reassessing: N}
> By priority: {1: N, 2: N, 3: N, 4: N, 5: N}
> {if any pending_review}: ⚠ {N} opportunities need priority decisions
>
> What would you like to do?

Available actions:
- **View/filter the list** → Step 3
- **Update an opportunity** → Step 4
- **Change priority** → Step 5
- **Change status** → Step 6
- **Handle pending reviews** → Step 7

---

### Step 3: View and Filter

The member can filter by:
- **Priority**: "Show me all priority 1 and 2 opportunities"
- **Status**: "Show me active opportunities" or "What's been deferred?"
- **Date range**: "Opportunities identified in the last 30 days" or "What hasn't been evaluated since January?"
- **Combination**: "Active priority 1 opportunities identified this quarter"

Present matching opportunities in a table format:

> | ID | Title | Priority | Status | Identified | Last Evaluated |
> |---|---|---|---|---|---|
> | OPP-001 | {title} | 1 | active | 2026-03-01 | 2026-03-20 |
> | OPP-003 | {title} | 2 | pursued | 2026-03-05 | 2026-03-20 |

Default sort: by priority (1 first), then by identified date (newest first). The member can request different sorting: "Sort by last evaluated" or "Oldest first."

After presenting, ask: "Want to see details on any of these, or make changes?"

If the member asks to see details on a specific opportunity, present the full record: ID, title, description, strategy connection, surfaced in briefing, priority (and who decided it), status, all three dates, and notes.

---

### Step 4: Update an Opportunity

If the member identified an opportunity: load it. If not: ask which one.

The member can update:
- **Title** — rename the opportunity
- **Description** — add detail or refine the description
- **Notes** — add freeform notes (appended, not replaced — notes accumulate over time)
- **Strategy connection** — update how this opportunity connects to the strategy

Present the changes for confirmation. On confirmation:
- Update the fields in `opportunities.json`
- Set `last_updated_date` to today
- Write the file

---

### Step 5: Change Priority

If the member identified an opportunity and a new priority: apply directly.

If not: ask which opportunity and what priority (1–5).

Present: "Change {OPP-XXX}: '{title}' from priority {old} to priority {new}?"

On confirmation:
- Update `priority` in `opportunities.json`
- Update `priority_decided_by` to the current member
- Set `last_updated_date` to today
- Write the file

The member can also bulk-change: "Set all deferred opportunities to priority 5" — present the list of affected opportunities and confirm before applying.

---

### Step 6: Change Status

Valid status transitions:

| From | Allowed To |
|---|---|
| pending_review | active, deferred, dismissed |
| active | pursued, deferred, dismissed, reassessing |
| pursued | active, deferred, dismissed, reassessing |
| reassessing | active, pursued, deferred, dismissed |
| deferred | active, dismissed |
| dismissed | active (reopen) |

If the member requests an invalid transition: explain what transitions are available from the current status.

Present: "Change {OPP-XXX}: '{title}' from {old status} to {new status}?"

On confirmation:
- Update `status` in `opportunities.json`
- Set `last_updated_date` to today
- If moving to `active` from `pending_review`: verify a priority has been set (it must have been — pending_review opportunities always need a priority decision)
- Write the file

The member can also bulk-change statuses: "Dismiss all priority 5 opportunities" — present the list and confirm.

---

### Step 7: Handle Pending Reviews

If there are opportunities with `pending_review` status, present them one at a time:

For each:
> **{OPP-XXX}: {title}**
> {description}
> Surfaced in briefing: {briefing_id}
> Suggested priority: {if available, from the briefing}
>
> What priority do you want to assign? (1 = highest, 5 = lowest)

Follow the same rules as the briefing priority decision phase — the member must give an explicit number. "Skip" is not accepted. If the member is truly not ready: "Would you like to defer this opportunity instead? You can come back to it later."

After all pending reviews are handled:
- Update each opportunity's `status` to `active`, `priority` to the decided value, `priority_decided_by` to the current member, and `last_updated_date` to today
- Write `opportunities.json`

---

## Directives

### Behavior

The opportunity registry should feel like a strategic dashboard, not a database query interface. When the member asks to see their opportunities, present them in a way that tells a story — not just data rows.

When presenting the dashboard, highlight what needs attention: pending reviews first, then recently surfaced opportunities, then anything that hasn't been evaluated in a while.

When the member changes a priority, don't question their judgment — but if they're making a significant change (e.g., priority 1 to priority 5), it's worth asking: "Want to add a note about why the priority changed? Helpful for future reference."

### Constraints

Never delete opportunities. Status changes are the only way to handle opportunities that are no longer relevant. `dismissed` is the terminal state for opportunities the member doesn't want to pursue.

Never change priorities or statuses without explicit member confirmation.

Never create new opportunities from this task. New opportunities come from briefing runs (`run-briefing`) and strategy reviews (`strategy-review`).

### Edge Cases

If the member asks to see opportunities from a specific briefing: filter by `surfaced_in_briefing` and present.

If the registry is very large (50+ opportunities): present the summary dashboard first and encourage filtering rather than listing everything.

If the member asks "what should I focus on?" — present active priority 1 and 2 opportunities, sorted by last evaluated date (oldest first, since those may need fresh evaluation).

If an opportunity has been `active` for a long time without evaluation: note it — "{OPP-XXX} hasn't been evaluated since {date}. You might want to review it in your next briefing or strategy review."
