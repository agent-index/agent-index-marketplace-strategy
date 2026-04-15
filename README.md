# agent-index-marketplace-strategy

A collection for building, evaluating, and evolving named strategies through agent-index. Define canonical strategy references, configure information sources, run briefings that surface opportunities, and maintain a prioritized opportunity registry — privately or shared with collaborators.

## Core Concepts

**Strategy** — a living canonical reference document defining vision, pillars, objectives, competitive landscape, constraints, and assumptions. Every edit is logged to an append-only changelog.

**Sources** — free-form information inputs (email labels, RSS feeds, web searches, publications — anything the member describes). Each source tracks its own cursor to prevent double-processing across briefing runs.

**Briefings** — the evaluation cycle. Pull unprocessed source material, evaluate it against the canonical strategy, surface new opportunities, and re-evaluate existing ones. Produces immutable dated documents.

**Opportunities** — a master registry of strategic opportunities tracked over time. Each has a numeric priority (1–5), a status lifecycle, and three key dates (identified, last evaluated, last updated). New opportunities require explicit priority decisions — no silent defaults.

## API Tasks

| Task | Description |
|---|---|
| `create-strategy` | Create a named strategy with canonical reference and full directory structure |
| `edit-strategy` | Modify the canonical reference with changelog tracking; warns non-owners |
| `manage-sources` | Add, edit, remove, and test information sources |
| `run-briefing` | Core cycle: pull sources, evaluate, surface opportunities, get priority decisions |
| `manage-opportunities` | View, filter, sort, update the opportunity registry |
| `share-strategy` | Promote private strategy to shared space, invite collaborators |
| `strategy-review` | Deep re-evaluation of opportunities without new source input |
| `strategy-tutorial` | End-user guide to the strategy system |

## Directory Structure

```
/{strategy-slug}/
  strategy.md                ← canonical reference (YAML frontmatter + structured body)
  sources.json               ← configured sources with cursors
  opportunities.json         ← master opportunity registry
  strategy-changelog.jsonl   ← append-only edit history
  /briefings/
    {YYYY-MM-DD}-{NNN}.md    ← immutable briefing snapshots
    {YYYY-MM-DD}-review-{NNN}.md  ← strategy review snapshots
  /state/
    current-context.md       ← rolling strategic context summary
```

## Sharing Model

Strategies start private in the member's **local** workspace (`members/{hash}/strategies/`, accessed via native file tools). When shared, they are promoted to the org's shared strategies path on the remote filesystem (default `/shared/strategies/`, configurable by org admin, accessed via `aifs_*` tools). Two roles: **owner** (controls canonical reference) and **collaborator** (can do everything else). Non-owners who edit the canonical reference are warned but not blocked — filesystem permissions are the real enforcement.

## Org Admin Setup

During collection install, the org admin configures:
- Shared strategies path (default `/shared/strategies/`)
- Default items per briefing run (default 10)
- Whether strategies must start private before sharing (`private_first` or `either`)

## Version

1.0.2
