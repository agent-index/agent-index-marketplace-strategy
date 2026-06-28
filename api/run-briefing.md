---
name: run-briefing
type: task
version: 1.2.0
collection: strategy
description: Pulls unprocessed items from configured sources, evaluates them against the canonical strategy reference, surfaces new opportunities with suggested priorities, re-evaluates existing opportunities, and requires explicit priority decisions before finalizing.
stateful: true
produces_artifacts: true
produces_shared_artifacts: false
dependencies:
  skills: []
  tasks: []
external_dependencies:
  - system: "Gmail / Outlook / other email provider (conditional)"
    access_required: "Read email messages by label or folder"
    contact: "org admin"
  - system: "Web search / RSS (conditional)"
    access_required: "Fetch web content and RSS feeds"
    contact: "org admin"
reads_from: null
writes_to: null
---

## About This Task

The briefing run is the core cycle of the strategy collection. It pulls unprocessed items from the strategy's configured sources, evaluates them against the canonical strategy reference, surfaces new opportunities, and re-evaluates existing active opportunities in light of new information.

The briefing produces an immutable dated document — a snapshot of what was found, how it relates to the strategy, and what opportunities it suggests. Once finalized, a briefing is never modified.

The critical behavioral requirement: every new opportunity surfaced by a briefing must have its priority explicitly decided by the member before the briefing is finalized. The system suggests a priority (1–5) with reasoning, but the opportunity enters `pending_review` status until the member actively accepts or changes the priority. No silent defaults.

### Inputs

The member identifies which strategy to brief. Optionally, the member can override the items-per-run cap for this specific run or select specific sources to pull from.

### Outputs

- `{strategy-path}/{slug}/briefings/{YYYY-MM-DD}-{NNN}.md` — the immutable briefing document
- Updated `sources.json` — advanced cursors for all processed sources
- Updated `opportunities.json` — new opportunities added, existing opportunities re-evaluated
- Updated `state/current-context.md` — refreshed strategic context

### Cadence & Triggers

Manual invocation only. The member runs briefings when they want to process new source material. If automation is desired, the member can schedule this through native Cowork scheduling.

---

## Workflow

### Step 1: Identify Strategy and Load Context

Read `collection-setup-responses.md` via `aifs_read` to get `default_items_per_run`. Read `member-index.json` (local) for `member_hash` and `member_folder_id`.

**Tool selection:** Operations on the member's local private workspace (`members/{member_hash}/strategies/`) use native Read/Write tools. Operations on **shared** strategies use `aifs_*` tools with **ID anchors** (standards.md § "Addressing"): your own shared strategies live at `id:{member_folder_id}/strategies/{slug}/`; strategies shared *with* you are discovered via the pointer index `/shared/strategies-index/` and opened via the cross-drive anchor `id:{item_drive_id}:{folder_id}/...` when the pointer carries `item_drive_id` (the strategy lives on the owner's drive — C.1.3 `crossdriveread`), falling back to the bare `id:{folder_id}/...` only for older pointers; the qualified form is OneDrive parity and harmless on gdrive (collaborators have write access and can run briefings into the shared copy).

If the member named a strategy: find it. If not: list available strategies and ask.

Read the following files from the strategy directory (using the appropriate tool based on location):
- `strategy.md` — the canonical reference (load fully into context)
- `sources.json` — the source registry with cursors
- `opportunities.json` — the current opportunity registry
- `state/current-context.md` — the rolling context summary

If `sources.json` has zero sources: halt with "This strategy has no sources configured. Add sources with '@ai:manage-sources' before running a briefing."

**On success:** Proceed to Step 2.

---

### Step 2: Configure This Run

Present the source list with their status:

> **Sources for '{strategy name}'**
> {for each source}:
> - {name}: {description} — Last processed: {date or "never"}, Items per run: {items_per_run}

Ask: "Run the briefing with all sources and default settings, or would you like to adjust anything for this run?"

The member can:
- **Run with defaults** — process all sources at their configured items-per-run cap
- **Select specific sources** — only pull from a subset
- **Override items per run** — process more or fewer items from any source for this run only
- **Override globally** — set a different cap for all sources this run

Record the run configuration. Do not modify `sources.json` — overrides apply to this run only.

**On success:** Proceed to Step 3.

---

### Step 3: Pull Source Material

For each source included in this run, in order:

1. Determine how to pull from it based on the source description. Use whatever tools and connectors are available — email MCP for email labels, web fetch for RSS and web pages, web search for search-based sources.

2. Pull items that are newer than the source's cursor position (`cursor.last_processed`). If the cursor is null (first run for this source), ask the member: "This is the first run for '{source name}'. How far back should I look? A date, a timeframe like 'last 2 weeks', or 'as far back as possible'?"

3. Respect the items-per-run cap. If more items are available than the cap, process the most recent items and note how many remain: "{N} items from '{source name}' were processed. {M} more are available for the next run."

4. For each item pulled, capture: the content (paraphrased, not verbatim), the source it came from, and the date/time of the original item.

5. Update the cursor for this source to reflect the most recent item processed. Store the new cursor data in memory — it will be written to `sources.json` in Step 8.

**If a source is unreachable:** Note the failure and continue with remaining sources. Record the failure: "Source '{name}' couldn't be reached — {reason}. Skipping for this run." Update `last_run_status` to `"failed: {reason}"` for this source.

**If no items are found across all sources:** Surface: "No new items found across any of your sources since the last briefing. Nothing to evaluate right now." Halt — do not create an empty briefing.

**On success:** Proceed to Step 4.

---

### Step 4: Evaluate Against Strategy

With the canonical strategy reference loaded and all source material collected:

1. **Summarize the source material** — organize by theme, not by source. Group related items together. Paraphrase — never reproduce source content verbatim. Identify the key signals: what's happening in the landscape that's relevant to this strategy.

2. **Evaluate relevance** — for each thematic group, assess how it relates to the strategy's vision, pillars, objectives, competitive landscape, constraints, and assumptions. Not everything pulled from sources will be relevant — separate signal from noise explicitly.

3. **Surface new opportunities** — identify potential strategic opportunities suggested by the source material. For each new opportunity:
   - Title: concise, specific
   - Description: what the opportunity is and why it matters to this strategy
   - Reasoning: how this connects to specific elements of the canonical strategy (reference specific pillars, objectives, or competitive dynamics)
   - Suggested priority: 1–5 (where 1 is highest) with explicit reasoning for the suggested level
   - Source attribution: which source items led to this opportunity

4. **Re-evaluate existing opportunities** — review the current `active`, `pursued`, and `reassessing` opportunities in `opportunities.json`. For each one that is affected by the new source material:
   - Explain how the new information affects the opportunity
   - Note whether the opportunity is strengthened, weakened, or changed
   - Suggest whether the priority should be adjusted
   - If an opportunity is no longer relevant, suggest moving to `dismissed`

**On success:** Proceed to Step 5.

---

### Step 5: Present Briefing to Member

Present the briefing in a structured format:

> **Strategy Briefing: {strategy name}**
> **Date:** {today}
> **Sources processed:** {N} items from {M} sources
> {if any sources failed}: **Sources skipped:** {list with reasons}
>
> ---
>
> ## Landscape Summary
> {thematic summary of source material and its relevance to the strategy}
>
> ---
>
> ## New Opportunities
> {for each new opportunity}:
> **{title}**
> {description}
> Connection to strategy: {reasoning}
> Suggested priority: {1-5} — {reasoning for priority level}
> Source: {attribution}
>
> ---
>
> ## Impact on Existing Opportunities
> {for each affected existing opportunity}:
> **{OPP-XXX}: {title}** (current priority: {N})
> {how new information affects this opportunity}
> Suggested action: {adjust priority / maintain / dismiss}
>
> {if no existing opportunities are affected}: No existing opportunities are affected by this briefing's findings.

Give the member time to read and process.

**On success:** Proceed to Step 6.

---

### Step 6: Priority Decisions

This step is mandatory. The briefing cannot be finalized until every new opportunity has an explicit priority decision.

For each new opportunity, in order:

Present: "**{title}** — I suggested priority {N}. {one-sentence reasoning}. What priority do you want to assign? (1 = highest, 5 = lowest)"

Accept the member's decision:
- If they agree with the suggestion: record the suggested priority as the decided priority
- If they provide a different priority: record their chosen priority
- If they want to discuss: engage — help them think through the priority level. But always come back to: "What priority do you want to assign?"

The member must give an explicit number (1–5) for each new opportunity. "Fine" or "sure" in response to the suggestion counts as agreement. "Skip" or "later" does not — surface: "Each opportunity needs a priority before I can finalize the briefing. If you're not sure, a middle-of-the-road 3 is a reasonable starting point — you can adjust later with '@ai:manage-opportunities'."

Also present any existing opportunities with suggested changes:

"**{OPP-XXX}: {title}** — currently priority {N}. Based on the new information, I suggest {action}. What do you think?"

For existing opportunities, the member can:
- Accept the suggestion
- Provide a different action
- Acknowledge without changing: "Noted, keep as-is"

Existing opportunity reviews do not block finalization — acknowledgment is sufficient.

**On success (all new opportunities have priorities):** Proceed to Step 7.

---

### Step 7: Confirm and Finalize

Present a final summary:

> **Briefing Summary**
> New opportunities: {N} (with priorities assigned)
> Existing opportunities affected: {M}
> {list new opportunities with their decided priorities}
>
> Finalize this briefing?

Wait for confirmation.

**On success:** Proceed to Step 8.

---

### Step 8: Write Everything

Determine the briefing number for today: check the briefings directory for existing files matching `{today}-*.md`. The new briefing is `{today}-{NNN}.md` where NNN is the next available number (001, 002, etc.).

1. Write the briefing document to `{strategy-path}/{slug}/briefings/{YYYY-MM-DD}-{NNN}.md`:

   ```
   ---
   briefing_id: "{YYYY-MM-DD}-{NNN}"
   strategy: "{strategy slug}"
   date: "{today YYYY-MM-DD}"
   author:
     display_name: "{display_name}"
     member_hash: "{member_hash}"
   sources_processed: {N}
   sources_skipped: {M}
   new_opportunities: {count}
   existing_opportunities_affected: {count}
   ---

   ## Landscape Summary

   {thematic summary}

   ## New Opportunities

   {for each new opportunity}:
   ### {title}

   {description}

   **Connection to strategy:** {reasoning}
   **Priority:** {decided priority} (suggested: {original suggestion})
   **Source:** {attribution}

   ## Impact on Existing Opportunities

   {for each affected opportunity}:
   ### OPP-{XXX}: {title}

   {how new information affects this opportunity}
   **Action taken:** {what the member decided}

   ## Sources Processed

   {for each source}:
   - {source name}: {items processed} items ({date range})
   {if skipped}: - {source name}: SKIPPED — {reason}
   ```

2. Update `opportunities.json`:

   For each new opportunity, add:
   ```json
   {
     "id": "OPP-{NNN}",
     "title": "{title}",
     "description": "{description}",
     "surfaced_in_briefing": "{briefing_id}",
     "status": "active",
     "priority": {decided priority},
     "priority_decided_by": {
       "display_name": "{display_name}",
       "member_hash": "{member_hash}"
     },
     "identified_date": "{today}",
     "last_evaluated_date": "{today}",
     "last_updated_date": "{today}",
     "strategy_connection": "{reasoning}",
     "notes": null
   }
   ```
   Increment `next_id` for each new opportunity. Update `last_updated`.

   For each existing opportunity that was affected: update `last_evaluated_date` to today. If the member changed the priority, update `priority`, `priority_decided_by`, and `last_updated_date`. If the member changed the status, update `status` and `last_updated_date`.

3. Update `sources.json`:
   For each source processed: update `cursor.last_processed` and `cursor.cursor_data` to reflect the most recent item processed. Set `last_run_status` to `"success"`. Update `last_updated`.

4. Update `state/current-context.md`:
   Regenerate to reflect the updated strategic position, incorporating the new briefing findings, new opportunities, and any changes to existing opportunities.

5. If the strategy is shared: no index update is needed — briefing results live in the strategy folder itself, and the pointer at `/shared/strategies-index/` is discovery-only. (There is no shared strategies-manifest in 1.1.0+.)

6. Append to `strategy-changelog.jsonl`:
   ```json
   {
     "timestamp": "{ISO 8601}",
     "type": "briefing_run",
     "author_hash": "{member_hash}",
     "author_name": "{display_name}",
     "briefing_id": "{briefing_id}",
     "new_opportunities": {count},
     "existing_affected": {count},
     "sources_processed": {count},
     "summary": "Briefing run: {count} new opportunities surfaced, {count} existing reviewed"
   }
   ```

7. Confirm to member:
   > "Briefing finalized. {N} new opportunities added to the registry, {M} existing opportunities updated. View the full briefing at `{briefing path}`. Manage opportunities anytime with '@ai:manage-opportunities'."

---

## Directives

### Behavior

The briefing run is the most important task in the collection. Take it seriously. The quality of the evaluation directly determines the value of the opportunity registry.

When evaluating source material against the strategy: be specific about connections. "This is relevant to your strategy" is not useful. "This directly affects your 'platform ecosystem' pillar because the new API marketplace reduces barrier to entry for third-party integrators" is useful.

When suggesting priorities, be honest about uncertainty. If the signal is weak, suggest a lower priority and say so: "This is early — it might be noise. I'd suggest priority 4 for now and see if future briefings reinforce the signal."

During the priority decision phase, respect the member's judgment. They know their strategic context better than the briefing data alone reveals. If they assign a different priority than suggested, accept it without resistance.

If a briefing run is interrupted (member needs to stop mid-run), save progress. Record which sources have been processed, which opportunities have been reviewed, and what's left. The member can resume by running the briefing again — the system should detect incomplete progress and offer to continue.

### Output Standards

Briefing documents are immutable once written. Never modify a past briefing.

Opportunity IDs are auto-incrementing and never reused. If OPP-005 is dismissed, OPP-005 stays as the dismissed entry. The next opportunity is OPP-006.

All JSON files must be valid after writing. Parse and serialize properly.

Paraphrase all source material in the briefing document. Never reproduce source content verbatim. The briefing is an analysis, not a clipping service.

### Constraints

Never finalize a briefing with opportunities in `pending_review` status. Every new opportunity must have an explicitly decided priority.

Never auto-promote opportunities from suggestions to the registry without the member's explicit priority decision.

Never modify `strategy.md` during a briefing run. The canonical reference is read-only during briefings. If the briefing suggests the strategy should change, note it in the briefing document and suggest the member run '@ai:edit-strategy' afterward.

Never skip the source cursor advancement. Even if source material turns out to be irrelevant, the cursor must advance to prevent re-processing.

### Edge Cases

If all sources fail (none reachable): halt with a clear explanation. Do not create an empty briefing.

If the member has 0 active opportunities and the briefing finds nothing new: "The sources had new material but nothing connected to your current strategy. This could mean the sources need adjustment or the strategy is well-covered. No new opportunities to add."

If the briefing surfaces a large number of opportunities (more than 10): warn the member up front — "This briefing surfaced {N} potential opportunities. That's a lot to prioritize in one session. Want to work through all of them now, or prioritize the most promising ones and save the rest for later?" If saving for later, create the remaining opportunities with `pending_review` status and note them for the next session.

If `opportunities.json` has opportunities in `pending_review` from a previous interrupted briefing: surface them at the start — "You have {N} opportunities from a previous briefing that still need priority decisions. Want to handle those first?"

If the briefing finds information that contradicts a strategy assumption: flag it prominently — "One of your stated assumptions may be challenged by this finding: {assumption}. Consider whether this requires a strategy update via '@ai:edit-strategy'."
