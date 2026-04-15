---
name: manage-sources
type: task
version: 1.0.2
collection: strategy
description: Add, edit, remove, and test information sources for a strategy. Sources are free-form — the member describes where to pull information and Claude handles the mechanics. Each source tracks its own cursor to prevent double-processing.
stateful: false
produces_artifacts: false
produces_shared_artifacts: false
dependencies:
  skills: []
  tasks: []
external_dependencies:
  - system: "Gmail / Outlook / other email provider (conditional)"
    access_required: "Read email messages by label or folder (only if email-based sources are configured)"
    contact: "org admin"
  - system: "Web search / RSS (conditional)"
    access_required: "Fetch web content and RSS feeds (only if web-based sources are configured)"
    contact: "org admin"
reads_from: null
writes_to: null
---

## About This Task

Sources are the information inputs that feed a strategy. They provide the raw material that briefings evaluate against the canonical strategy reference. This task manages the source registry — adding new sources, editing existing ones, removing sources that are no longer relevant, and testing whether a source is reachable.

Sources have no rigid type system. A source is whatever the member describes: an email label, an RSS feed, a web search query, a specific publication, a competitor's blog, an industry report series. The member describes the source in natural language, and Claude determines how to pull from it during briefing runs.

Each source maintains its own cursor — a checkpoint that tracks what has already been processed. This prevents items from being evaluated twice across briefing runs.

### Inputs

The member identifies which strategy to manage sources for and describes what they want to do (add, edit, remove, test, or list sources).

### Outputs

Updated `sources.json` for the identified strategy.

### Cadence & Triggers

On demand, whenever the member wants to configure their information inputs.

---

## Workflow

### Step 1: Identify Strategy

Read `collection-setup-responses.md` via `aifs_read` to get `shared_strategies_path` and `default_items_per_run`.

**Tool selection:** Operations on the member's private workspace (`/members/{member_hash}/strategies/`) use native Read/Write tools. Operations on the shared strategies path (`{shared_strategies_path}`) use `aifs_*` tools (e.g., `aifs_read`, `aifs_write`, `aifs_exists`).

If the member named a strategy: find it in the member's private workspace or shared space.

If the member did not name a strategy: list available strategies and ask.

Read `sources.json` for the identified strategy (using the appropriate tool based on location).

**On success:** Proceed to Step 2.

---

### Step 2: Determine Action

If the member stated what they want to do in their invocation (e.g., "add a source to my AI strategy"), proceed to the appropriate action.

If the member's intent is unclear, present the current source list (or note "No sources configured yet") and ask: "What would you like to do?"

Available actions:
- **Add a source** → Step 3
- **Edit an existing source** → Step 4
- **Remove a source** → Step 5
- **Test a source** → Step 6
- **List sources** → Step 7

---

### Step 3: Add a Source

Ask: "Describe the source — where should I pull information from? For example: 'My Gmail label called Industry News,' 'The Stratechery blog RSS feed,' 'Search for news about [competitor name],' 'Articles from the Harvard Business Review about AI strategy.'"

Accept the member's description. Do not force them into a type taxonomy — record exactly what they describe.

Ask: "What should I call this source? A short name for the list." If the member doesn't have a preference, suggest one based on the description (e.g., "Industry News Emails," "Stratechery RSS," "Competitor Watch").

Ask: "How many items should I process from this source per briefing run? The default is {default_items_per_run}." Accept any positive integer or "default."

Present the source for confirmation:

> **New Source**
> Name: {name}
> Description: {description}
> Items per run: {items_per_run}
>
> Add this source?

On confirmation, add to `sources.json`:
```json
{
  "name": "{name}",
  "description": "{description}",
  "items_per_run": {items_per_run},
  "cursor": {
    "last_processed": null,
    "cursor_data": {}
  },
  "added_date": "{today}",
  "added_by": {
    "display_name": "{display_name}",
    "member_hash": "{member_hash}"
  },
  "last_run_status": null
}
```

Update `last_updated` on `sources.json`. Write the file.

Confirm: "Source '{name}' added. You now have {N} source(s) configured."

Ask: "Would you like to add another source, or are you done?"

---

### Step 4: Edit a Source

If the member named a source: find it in `sources.json`.

If not: present the source list and ask which one to edit.

Present the current source details and ask what the member wants to change. The member can update:
- **Name** — the display name
- **Description** — the source description (what to pull from)
- **Items per run** — the per-source cap

The member cannot directly edit cursor data — cursors are managed by the briefing run.

Confirm the changes, update `sources.json`, and write.

If the member changes the description significantly (e.g., changing from one email label to a completely different source), ask: "This is a substantial change. Should I reset the cursor so the next briefing run starts fresh for this source, or keep the existing checkpoint?" If reset: set `cursor.last_processed` to `null` and `cursor.cursor_data` to `{}`.

---

### Step 5: Remove a Source

If the member named a source: find it in `sources.json`.

If not: present the source list and ask which one to remove.

Confirm: "Remove source '{name}'? This won't affect opportunities that were already surfaced from this source — they stay in the registry."

On confirmation: remove the entry from `sources.json`, update `last_updated`, write the file.

---

### Step 6: Test a Source

If the member named a source: find it in `sources.json`.

If not: present the source list and ask which one to test.

Attempt to reach the source based on its description:
- For email-based sources: check if the email connector is available and the label/folder exists. Report how many unread/recent items are available.
- For RSS/web sources: attempt to fetch the feed or page. Report whether it's reachable and show a sample of recent items.
- For search-based sources: run the described search and show a sample of results.
- For sources Claude can't automatically test (manual sources, obscure references): explain that automatic testing isn't available and suggest the member verify access manually.

Report the result: "Source '{name}' is reachable. I can see {N} recent items." or "I can't reach this source right now — {reason}."

A failed test does not remove the source. The member may want to keep a source configured even if it's temporarily unreachable.

---

### Step 7: List Sources

Present all configured sources with:
- Name
- Description (truncated if long)
- Items per run
- Last processed date (or "Never" if cursor is null)
- Last run status (or "Never run" if null)

If no sources are configured: "No sources configured yet. Would you like to add one?"

---

## Directives

### Behavior

Sources are intentionally free-form. The member shouldn't feel constrained to a set of predefined types. If they say "check the front page of Hacker News for AI-related stories" or "pull from the #strategy channel in our Slack" or "review the latest McKinsey quarterly report" — accept all of these as valid source descriptions. Claude determines how to pull from them during the briefing run.

When helping the member add sources, be proactive about suggesting related sources: "You mentioned watching TechCrunch newsletters — would you also want to track their RSS feed for articles that don't make the newsletter?"

Keep the source list manageable. If a member is adding their 15th source, note: "You've got quite a few sources now. More sources means longer briefing runs — each one adds items to process. You might consider consolidating similar sources."

### Constraints

Never modify cursor data directly through this task. Cursors are advanced by `run-briefing` only. The one exception: resetting a cursor when a source description changes substantially (Step 4).

Never auto-remove sources based on test failures. Sources may be temporarily unreachable.

Never modify `opportunities.json` or `strategy.md` from this task.

### Edge Cases

If `sources.json` doesn't exist (e.g., strategy was created with an older version): initialize it with the standard empty structure before proceeding.

If the member adds a source that sounds identical to an existing one: note the overlap — "This looks similar to your existing source '{name}'. Want to update that one instead, or is this intentionally separate?"

If the member wants to add a source that requires a connector Claude doesn't currently have access to (e.g., Gmail but no email MCP): accept the source anyway, but note: "I don't have access to Gmail right now. The source is saved, but I won't be able to pull from it during briefings until the email connector is set up. Your org admin can help with that."
