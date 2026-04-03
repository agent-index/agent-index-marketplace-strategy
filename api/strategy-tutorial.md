---
name: strategy-tutorial
type: skill
version: 1.0.2
collection: strategy
description: Explains the strategy collection to members — its concepts, workflows, and how to be productive with it — through a guided tour or targeted answers to specific questions. Covers strategy creation, sources, briefings, opportunities, sharing, and reviews.
stateful: false
always_on_eligible: false
dependencies:
  skills: []
  tasks: []
external_dependencies: []
---

## About This Skill

The strategy collection is a system for building, evaluating, and evolving named strategies. It involves several interconnected concepts — canonical references, sources, briefings, opportunities, sharing, and reviews — that members encounter gradually as they work.

The Strategy Tutorial Skill is how questions about the system get answered. It serves two modes. In guided tour mode, it walks a member through the system from first principles. In question-answering mode, it responds to specific questions.

This skill explains — it does not perform operations.

### When This Skill Is Active

When invoked, Claude shifts into explanatory mode, drawing on knowledge of the strategy collection to answer questions and guide members. The skill remains active for the tutorial conversation.

### What This Skill Does Not Cover

This skill covers the strategy collection's concepts and workflows. It does not cover the broader agent-index system (that's the System Tutorial in agent-index-core). It does not troubleshoot remote filesystem issues. It does not cover internal file format details — the task definition files serve that purpose.

---

## Directives

### Behavior

When invoked, determine whether the member wants a guided tour or has a specific question. A guided tour is indicated by phrases like "show me how strategy works," "walk me through it," or a bare invocation. A specific question is anything more targeted.

For a guided tour: run the structured tour sequence. Check in after each topic.

For a specific question: answer directly. Offer to go deeper afterward.

Read the member's `member-index.json` and the org's `collection-setup-responses.md` for the strategy collection before responding, so examples reflect their actual configuration.

### Guided Tour Sequence

Seven topics in order. After each, check in: "Does that make sense? Want to go deeper, or shall we move on?"

**Topic 1: What the strategy collection does**

The strategy collection gives you a structured way to build and evolve strategies through Claude. Instead of keeping strategy in your head or in static documents that go stale, you define a living strategy document and then systematically evaluate information against it over time.

The system has a core loop: you define a strategy, you tell it where to find relevant information (sources), you run briefings that evaluate that information against your strategy, briefings surface opportunities, and you track those opportunities over time. Some opportunities may even cause you to update the strategy itself. Then the cycle repeats.

You can have multiple strategies. Each one is independent — its own document, its own sources, its own opportunities. A company might have a product strategy, a market expansion strategy, and a talent strategy, each running their own briefing cycles.

**Topic 2: The canonical strategy reference**

Every strategy starts with a canonical reference document — the living definition of what the strategy actually is. When you create a strategy with `@ai:create-strategy`, Claude walks you through defining the key sections: vision, strategic pillars, objectives, competitive landscape, constraints, and assumptions.

This document is important because everything else in the system is evaluated against it. When a briefing processes news or information, it asks: "How does this relate to the strategy as defined?" When opportunities are prioritized, the question is: "How important is this to what the strategy is trying to achieve?"

The canonical reference is meant to evolve. As you learn new things, as the market shifts, as opportunities reshape your thinking — you update the strategy with `@ai:edit-strategy`. Every edit is logged to a changelog so you can always trace how your strategic thinking developed over time.

**Topic 3: Sources — where information comes from**

Sources are the information inputs that feed your strategy. You configure them with `@ai:manage-sources`. A source can be anything — an email newsletter label, an RSS feed, a web search query, a specific publication, a competitor's blog. You describe it in natural language and Claude figures out how to pull from it.

The key feature of sources is cursor tracking. Each source remembers what's already been processed, so when you run a briefing, it only looks at new material since the last run. You never process the same information twice.

You can add as many sources as you want, though more sources means longer briefing runs. Each source has its own items-per-run cap so you can control how much gets processed in a single session.

**Topic 4: Briefings — the evaluation cycle**

Briefings are the core activity. When you run `@ai:run-briefing`, Claude pulls new items from your sources, evaluates them against your canonical strategy, and produces a structured analysis.

The briefing organizes source material by theme, identifies what's relevant to your strategy and what's noise, and surfaces opportunities — things you might want to act on, explore, or keep an eye on. It also looks at your existing opportunities and flags any that are affected by the new information.

Each briefing is saved as a dated, immutable document. It's a snapshot of what was known at that point. You can look back at past briefings to see how the landscape evolved.

The most important part of a briefing is the priority decision phase. When the briefing surfaces new opportunities, Claude suggests a priority for each one (1 through 5, where 1 is highest). But the system doesn't just assign the suggestion and move on. You must explicitly decide — agree with the suggestion or set a different priority. Every opportunity needs your active input before the briefing is finalized. This prevents the registry from filling up with things nobody actually thought about.

**Topic 5: Opportunities — the strategic radar**

Opportunities are the long-term memory of the system. Every opportunity surfaced by a briefing goes into a master registry that you manage with `@ai:manage-opportunities`.

Each opportunity tracks three dates: when it was identified, when it was last evaluated (by a briefing or review), and when it was last updated (priority changed, notes added, etc.). Combined with the priority (1–5) and status, this gives you a complete picture of your strategic landscape.

Statuses follow a general pattern: an opportunity starts as `pending_review` until you assign a priority, then becomes `active`. From there it can move to `pursued` (you're acting on it), `deferred` (not now, but not dismissed), or `dismissed` (no longer relevant). Opportunities may also move to `reassessing` when you edit the strategy — this flags them for fresh evaluation against your updated direction. Opportunities are never deleted — even dismissed ones stay in the registry for historical reference.

You can filter and sort the registry by any combination of priority, status, and dates. "Show me all active priority 1 opportunities" or "What hasn't been evaluated in the last month?" gives you exactly the view you need.

**Topic 6: Private and shared strategies**

Strategies start in your private workspace. Your strategic thinking, your sources, your opportunities — all private until you decide otherwise.

When you're ready to collaborate, `@ai:share-strategy` promotes the strategy to the shared space. You invite collaborators who can then run briefings, manage sources, and evaluate opportunities alongside you.

The sharing model is simple: you're the owner, everyone else is a collaborator. Collaborators can do everything — the one guardrail is the canonical strategy reference. If a collaborator tries to edit the strategy document itself, the system warns them that changes to the canonical reference should go through the owner. It doesn't block them — but it makes the governance visible.

This keeps the canonical reference coherent. Multiple people can contribute to the information gathering and opportunity evaluation, but the strategic direction remains owned.

**Topic 7: Strategy reviews — stepping back**

Briefings are about processing new information. Reviews are about stepping back and looking at the full picture. `@ai:strategy-review` re-evaluates all your opportunities against the current strategy without pulling any new source material.

This is especially valuable after editing the strategy — if you change a pillar or adjust an objective, some opportunities may no longer align while others may become more important. A review catches these shifts.

Reviews are also good on a regular cadence — monthly or quarterly — to make sure your opportunity registry reflects your current thinking rather than accumulated historical findings.

After the tour: "Those are the core concepts. The daily workflow is straightforward: configure your sources once, then periodically run briefings to process new information and surface opportunities. Reviews keep the whole picture aligned. Your main commands are `@ai:create-strategy`, `@ai:manage-sources`, `@ai:run-briefing`, and `@ai:manage-opportunities`. Questions about any of it?"

### Answering Specific Questions

Common patterns:

**"How do I {accomplish something}?"** — Name the `@ai:` command and briefly explain.

**"What happens when {thing}?"** — Trace the flow.

**"What's the difference between {A} and {B}?"** — Draw clear distinctions. "Briefings vs. reviews: briefings process new source material and surface new opportunities. Reviews re-evaluate existing opportunities against the current strategy without new input."

**"Can I {do something}?"** — Honest answer. If yes, explain how. If no, explain alternatives.

### Style & Tone

Practical, concrete, patient. Not a manual. Use realistic examples — "Imagine you're running a market expansion strategy and your newsletter sources surface news about a competitor entering your target market..."

Avoid implementation details unless they help understanding. Members don't need to know about JSONL files or cursor mechanics. They need to know what they can do and how.

### Constraints

Do not perform operations while in tutorial mode. Direct the member to the appropriate `@ai:` command.

Do not speculate about behavior. If something doesn't match expectations, surface the discrepancy honestly.

Do not provide deep technical details about file formats or directory structures.

### Edge Cases

If the member asks about features they don't have configured: explain the capability and note it's not currently active.

If confused or frustrated: slow down, rebuild from the last clear concept.

If invoked mid-task: provide a brief targeted answer without disrupting the task context.
