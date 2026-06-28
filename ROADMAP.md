# Strategy Collection — Roadmap

Current version: 1.0.2
Last updated: 2026-04-05

---

## Current State

v1.0 provides the full strategy lifecycle: create canonical strategy references, configure information sources, run briefings that evaluate sources against the strategy, surface and prioritize opportunities, share strategies with collaborators, and run deep reviews without new source input. A guided tutorial helps new members onboard.

### Known Limitations

- **Source cursors are per-source, not per-briefing.** If a briefing run is interrupted partway through, the cursors for already-processed sources have advanced but the briefing document is incomplete. Re-running produces a briefing that skips the already-consumed items. A transactional cursor model (commit all cursors at briefing completion) would be more robust.
- **No source health monitoring.** If a source consistently returns no new material (dead RSS feed, search query with no results), the collection doesn't flag it. Members discover stale sources only when they notice briefings aren't surfacing anything new.
- **Opportunity priority is a static number.** Priorities (1-5) don't change automatically based on new information. Strategy reviews can adjust them, but there's no signal-driven re-prioritization between reviews.
- **Sharing is all-or-nothing.** When a strategy is shared, the entire canonical reference and opportunity registry become visible to collaborators. There's no way to share the strategy reference while keeping certain opportunities private.

### Known Bugs

None currently tracked.

---

## Wishlist

### v1.1 — Quality of Life

- **Source health checks.** Flag sources that have returned no new material for N consecutive briefing runs. Suggest removal or replacement.
- **Opportunity aging.** Automatically flag opportunities that haven't been evaluated in a configurable number of days/briefings.
- **Briefing diff.** When running a briefing, optionally show what changed since the last briefing — new opportunities, priority changes, status transitions.

### v1.2 — Cross-Collection Integration

- **Projects collection integration.** When an opportunity is promoted to "active" status, offer to create a project in the Projects collection (if installed) to track execution.
- **Capture collection integration.** Allow capturing items from the Capture collection as strategy sources or opportunity references.

### v2.0 — Structural Changes (breaking)

- **Multi-strategy comparison.** Compare opportunity registries across multiple strategies to identify overlap, conflicts, or synergies. Requires rethinking the per-strategy directory model.
- **Transactional source cursors.** Commit cursor advances only when a briefing completes successfully. Requires changes to the source and briefing data models.

---

## Design Notes

- Strategies start private by design. The `private_first` vs `either` configuration exists because some orgs want members to develop their strategic thinking privately before sharing, while others want collaborative strategy from day one. Neither is wrong — the collection supports both.

- Briefings are immutable snapshots. Once written, a briefing document is never modified. This creates an auditable trail of what was evaluated and when. The trade-off is storage growth, but briefings are small text files and the practical limit is far beyond what any org would hit.

- The collection deliberately avoids automated decision-making. Opportunities require explicit priority decisions from the member — no silent defaults, no auto-ranking. This is a design choice: strategic prioritization is inherently a human judgment call.
