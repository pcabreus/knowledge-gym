# AGENTS.md — Knowledge Gym (Static Markdown First)

## Mission
Build and maintain a **static knowledge base** for long-term learning and interview readiness. The repository is a **graph of topics**: no duplication, only references/links.

## Core principles (non-negotiable)
1. **No duplicate explanations.** If content already exists in another topic, link to it instead of rewriting.
2. **One canonical topic per concept.** If a user adds a term that already exists, create an alias page in `topics/_aliases/` that points to the canonical file.
3. **Graph-first navigation.** Every topic must include:
   - prerequisites (links),
   - related topics (links),
   - “part-of / compares-to” relationships where relevant.

## Repository conventions
- All docs are Markdown.
- Primary topic directory: `topics/`
- Topic filenames use **kebab-case**: `gin-index.md`, `event-sourcing.md`
- Topic location follows taxonomy in `docs/04-taxonomy.md`.
- Index pages:
  - `topics/_index.md` must list topics by category and be updated when new topics are added.
  - `questions/_index.md` must list question sets if any are added.

## Required topic structure
Use `templates/topic.template.md`. Not all sections must be filled; keep empty sections with “N/A” only if truly not applicable.

Mandatory minimum for every new topic:
- Definition (What it is)
- Why it matters
- When to use / when not to use
- Trade-offs (at least 2)
- Links: prerequisites + related
- At least 3 “trap questions” with answers

## Linking rules (avoid duplication)
- Prefer relative links: `../database/index.md`
- Link the first time a key term appears.
- If you mention a concept that has a topic file, **always link it**.

## Quality bar
Follow `docs/05-quality-bar.md`. Prioritize correctness, decision rules, and failure modes.

## Prompts
- To create a new topic from a raw term, use: `prompts/A-topic-intake.prompt.md`
- To create MCQs: `prompts/B-mcq-generator.prompt.md`
- To evaluate open answers: `prompts/C-open-answer-evaluator.prompt.md`

## Typical agent tasks
1. Add a new topic:
   - Determine canonical name + file path (taxonomy)
   - Create topic using template
   - Update `topics/_index.md`
   - Add alias file if needed
   - Add links to prerequisites/related topics
2. Improve existing topic:
   - Add missing trade-offs, failure modes, examples
   - Add “trap questions”
   - Ensure no duplicated explanation; replace with links

## Output expectations for agent
- Always propose file edits as unified diffs or full-file content.
- Never introduce new folders without updating docs accordingly.
- Never change conventions without updating `docs/02-authoring-guide.md`.
