# Knowledge Gym — Long-term technical learning (Static First)

This repo is my **living learning system** to master software engineering concepts and retain them long term.

## What it solves
1) **Deep learning**: each term/topic has a standardized card (“Knowledge Card”) with definition, uses, non-uses, trade-offs, traps, and mini-examples.
2) **Long-term retention**: the content is designed for practice with questions, evaluation, and periodic review (later: gamification + scheduling).

## How it’s used today (version 1: static Markdown)
- Topics live in `topics/` organized by categories.
- Each topic follows a common structure (see `templates/topic.template.md`).
- No duplicated explanations: navigate with links between concepts.
- Questions and sets (if any) live in `questions/`.

## Philosophy: “Graph, not encyclopedia”
A concept depends on others. Example:
- `GIN index` depends on `index`, `PostgreSQL`, `JSONB`.
Instead of repeating index or JSONB theory, we link to those topics.

## Repo structure
- `docs/`: vision, rules, taxonomy, authoring guide
- `topics/`: topic cards
- `questions/`: question banks
- `templates/`: templates
- `prompts/`: strict prompts to generate and evaluate content

## Next steps (version 2+)
- Practice engine (CLI or web)
- Spaced repetition (SM-2)
- Tracking results and “living importance”

See `ROADMAP.md`.

## Key conventions
- No duplication: if a topic exists, link it.
- One concept = one canonical file.
- Aliases in `topics/_aliases/`.

> Note: Explanations in English, with technical terms in English (parentheses) the first time they appear.
