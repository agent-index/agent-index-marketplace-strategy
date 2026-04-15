---
name: create-strategy
type: task
version: 1.0.2
collection: strategy
description: Creates a new named strategy with a canonical reference document, empty source and opportunity registries, and the full directory structure — in the member's private workspace or the shared space depending on org configuration.
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

Creating a strategy establishes the canonical reference document — the living definition that all briefings, opportunity evaluations, and strategic decisions are measured against. This task guides the member through defining the strategy's name, vision, pillars, objectives, competitive landscape, constraints, and assumptions, then writes the full directory structure.

Where the strategy is created depends on org configuration. If the org requires a private stage, the strategy is created in the member's private workspace at `/members/{member_hash}/strategies/{strategy-slug}/`. If the org allows either, the member chooses. Strategies created privately can later be shared via `@ai:share-strategy`.

### Inputs

The member provides: strategy name and the canonical reference content — vision, strategic pillars, objectives, competitive landscape, constraints, and assumptions. All sections are optional at creation time.

### Outputs

- `{strategy-path}/{strategy-slug}/strategy.md` — the canonical reference document
- `{strategy-path}/{strategy-slug}/sources.json` — empty source registry
- `{strategy-path}/{strategy-slug}/opportunities.json` — empty opportunity registry
- `{strategy-path}/{strategy-slug}/strategy-changelog.jsonl` — initial changelog entry
- `{strategy-path}/{strategy-slug}/briefings/` — empty briefings directory
- `{strategy-path}/{strategy-slug}/state/current-context.md` — initial context file

For shared strategies, also updates `strategies-manifest.json`.

### Cadence & Triggers

Run once per strategy at inception. Not repeatable for the same strategy — if a strategy already exists at the generated slug, this task surfaces that and offers to open `edit-strategy` instead.

---

## Workflow

### Step 1: Read Org Configuration

Read `collection-setup-responses.md` via `aifs_read` from the collection's setup directory. Extract:
- `shared_strategies_path` — where shared strategies live (on the remote filesystem)
- `strategies_require_private_stage` — `private_first` or `either`
- `default_items_per_run` — default briefing cap

Identify the running member's `member_hash` and `display_name` from session context.

**Tool selection:** Operations on the member's private workspace (`/members/{member_hash}/strategies/`) use native Read/Write tools. Operations on the shared strategies path (`{shared_strategies_path}`) use `aifs_*` tools (e.g., `aifs_read`, `aifs_write`, `aifs_exists`).

**On failure to read setup responses:** Check `aifs_auth_status()`. If `authenticated: false`, attempt automatic re-authentication via `aifs_authenticate` and retry the read. If re-auth fails or the read still fails: surface "The Strategy collection setup information couldn't be read. I tried to restore your connection but wasn't able to. Try '@ai:member-bootstrap' to troubleshoot, or contact your org admin." Halt.

**On success:** Proceed to Step 2.

---

### Step 2: Determine Location

**If `strategies_require_private_stage` is `private_first`:**
Inform the member: "Strategies in your org start as private drafts. I'll create this in your personal workspace. When you're ready to collaborate, you can share it with '@ai:share-strategy'."
Set `strategy_path` to `/members/{member_hash}/strategies/`.

**If `strategies_require_private_stage` is `either`:**
Ask: "Would you like to create this strategy privately (just for you, shareable later) or directly in the shared space (visible to collaborators immediately)?"
- If private: set `strategy_path` to `/members/{member_hash}/strategies/`
- If shared: set `strategy_path` to `{shared_strategies_path}`

**On success:** Proceed to Step 3.

---

### Step 3: Collect Strategy Name

Ask: "What is the name of this strategy?"

Accept any non-empty string. Generate the slug: lowercase, spaces and special characters replaced with hyphens, consecutive hyphens collapsed, leading and trailing hyphens removed.

Show the member the generated slug: "I'll use `{slug}` as the strategy's directory name."

Check whether `{strategy_path}/{slug}/` already exists. If it does:

**On conflict:** Surface: "A strategy with the slug `{slug}` already exists at this location. Would you like to edit that strategy instead, or choose a different name?" Do not proceed until resolved.

**On success:** Proceed to Step 4.

---

### Step 4: Collect Canonical Reference Content

Introduce the reference: "Let's define the strategy. I'll walk you through a few sections — skip any you're not ready for, you can always add them later with '@ai:edit-strategy'."

Guide the member through each section in order:

| Section | Prompt |
|---|---|
| **vision** | "What's the strategic vision? This is the high-level direction — where this strategy is heading and why it matters." |
| **pillars** | "What are the strategic pillars? These are the key themes or focus areas that support the vision. For example: 'product-led growth,' 'enterprise expansion,' 'platform ecosystem.'" |
| **objectives** | "What are the specific objectives? Concrete goals this strategy aims to achieve — measurable where possible." |
| **competitive_landscape** | "What does the competitive landscape look like? Who are the key players, what are they doing, and where do you see differentiation opportunities?" |
| **constraints** | "Are there any known constraints? Budget, timeline, regulatory requirements, resource limits, technology limitations?" |
| **assumptions** | "What assumptions is this strategy built on? Things that must be true for the strategy to work — market conditions, technology trends, customer behavior." |

For each section:
- If the member provides content, record it.
- If the member skips, record as `null`.
- If the member provides content for multiple sections at once, capture what was given and only ask about remaining sections.

**On success:** Proceed to Step 5.

---

### Step 5: Confirm and Write

Present a complete summary:

> **New Strategy**
> Name: {name}
> Slug: {slug}
> Location: {private workspace / shared space}
> Owner: {member display_name}
>
> Sections defined: {list sections with content}
> Sections skipped: {list sections without content}
>
> Ready to create this strategy?

Wait for explicit confirmation before writing anything.

On confirmation:

1. Create the strategy directory structure:
   ```
   {strategy_path}/{slug}/
     /briefings/
     /state/
   ```

2. Write `strategy.md`. Full format:

   ```
   ---
   name: {name}
   slug: {slug}
   owner:
     display_name: {display_name}
     member_hash: {member_hash}
     email: {email}
   collaborators: []
   created: {today YYYY-MM-DD}
   last_updated: {today YYYY-MM-DD}
   shared: {true if created in shared space, false if private}
   shared_path: null
   ---

   ## Vision

   {vision content or "*Not yet defined.*"}

   ## Strategic Pillars

   {pillars content or "*Not yet defined.*"}

   ## Objectives

   {objectives content or "*Not yet defined.*"}

   ## Competitive Landscape

   {competitive_landscape content or "*Not yet defined.*"}

   ## Constraints

   {constraints content or "*Not yet defined.*"}

   ## Assumptions

   {assumptions content or "*Not yet defined.*"}
   ```

3. Initialize `sources.json`:
   ```json
   {
     "last_updated": "{today}",
     "sources": []
   }
   ```

4. Initialize `opportunities.json`:
   ```json
   {
     "last_updated": "{today}",
     "next_id": 1,
     "items": []
   }
   ```

5. Initialize `strategy-changelog.jsonl` with the creation entry:
   ```json
   {
     "timestamp": "{ISO 8601}",
     "type": "strategy_created",
     "author_hash": "{member_hash}",
     "author_name": "{display_name}",
     "summary": "Strategy '{name}' created by {display_name}",
     "sections_defined": ["{list of sections with content}"]
   }
   ```

6. Write `state/current-context.md`:
   ```
   # {name} — Current Context
   **Last Updated:** {today}

   ## Strategic Position
   {If vision was provided: one-sentence summary. Otherwise: "No strategic position defined yet."}

   ## Active Opportunities
   None yet. Run '@ai:run-briefing' after configuring sources to begin surfacing opportunities.

   ## Recent Activity
   Strategy created {today}.
   ```

7. If created in shared space: read `strategies-manifest.json` at `{shared_strategies_path}` via `aifs_read`, add entry:
   ```json
   {
     "slug": "{slug}",
     "name": "{name}",
     "owner": "{owner display_name}",
     "owner_hash": "{owner member_hash}",
     "created": "{today YYYY-MM-DD}",
     "last_briefing": null,
     "opportunity_count": 0
   }
   ```
   Update `last_updated` on the manifest. Write the file via `aifs_write`.

8. Confirm to member:
   > "Strategy '{name}' has been created. Next step: add sources with '@ai:manage-sources' so you have information flowing into your briefings. When you're ready, run '@ai:run-briefing' to evaluate source material against your strategy."
   > {if private}: "This strategy is private. When you're ready to collaborate, share it with '@ai:share-strategy'."

**On any write failure:** Surface the specific file that failed, leave successfully written files in place, advise the member to retry.

---

## Directives

### Behavior

Collect all information conversationally before writing anything. The Step 5 confirmation is the single gate before any filesystem writes.

The canonical reference sections are the most important part. Help the member think through each one — offer prompts and examples, but don't be prescriptive. "For strategic pillars, some strategies organize around 3-5 themes — like 'product innovation,' 'market expansion,' 'operational efficiency.' What are the key themes for yours?"

If the member provides a rich narrative that covers multiple sections at once, parse it into the appropriate sections rather than asking redundant questions.

The strategy document is meant to be a living reference, not a perfect artifact at creation. Encourage the member to start with what they know and refine over time: "You can always update any section with '@ai:edit-strategy' as your thinking evolves."

### Output Standards

`strategy.md` must always be written with complete YAML frontmatter — no optional fields omitted. Skipped sections are written as `*Not yet defined.*` in the body, not omitted.

All JSON files must be valid JSON after writing. Always parse and serialize properly.

### Constraints

Never write to the filesystem before the Step 5 confirmation.

Never generate a slug that collides with an existing strategy directory.

If `strategies_require_private_stage` is `private_first`, never create a strategy directly in the shared space regardless of what the member requests. Explain: "Your org requires strategies to start as private drafts. You can share it with '@ai:share-strategy' when you're ready."

### Edge Cases

If the member provides a strategy name that generates a slug already in use: surface the conflict and offer alternatives.

If `strategies-manifest.json` does not exist at the shared path when creating a shared strategy: surface: "The strategies manifest file is missing. This usually means the collection setup wasn't completed. Ask your org admin to run the Strategy collection setup." Halt.

If the member provides a strategy name with only special characters that produce an empty slug: ask for a different name.

If the member's private strategies directory doesn't exist yet: create `/members/{member_hash}/strategies/` before proceeding.
