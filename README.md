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
| `share-strategy` | Share a strategy from your member space — readers, collaborators, or org-wide read |
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

Strategies start private in the member's **local** workspace (`members/{hash}/strategies/`, accessed via native file tools). When shared, the content is copied to the **owner's private member space on the remote filesystem** (`id:{member_folder_id}/strategies/{slug}/`, ID-anchor addressing) and access is granted per person via `permission-change-helper`: *share with X* = X can read; *make X a collaborator* = X can read + write; *share with the org* = everyone (`all@`) can read. Nobody else can see it. Discovery happens through a pointer index at `/shared/strategies-index/` — one small JSON pointer per shared strategy (owner, slug, folder_id, scope); the content itself never lives under `/shared`. Owner-vs-collaborator governance over the canonical reference remains task-level soft enforcement; the per-person grants are the real filesystem enforcement. Unshare/delete follow the core soft-delete conventions (revoke grants, mark pointer `revoked`/strategy `archived` — never trash).

## Org Admin Setup

During collection install, the org admin configures:
- Default items per briefing run (default 10)
- Whether strategies must start private before sharing (`private_first` or `either`)

Setup also ensures `/shared/strategies-index/` exists, and `install-collection` Step 5.5 applies the `collaborative-acls.json` grant (`all@` writer on the index folder). Requires agent-index-core 3.8.0+ and filesystem adapter 2.5.0+.

## Version

1.1.4
