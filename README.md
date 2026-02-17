# Knowledge Gym — Long-term technical learning (Static First)

This repo is my **living learning system** to master software engineering concepts and retain them long term.

## What it solves
1) **Deep learning**: each term/topic has a standardized card (“Knowledge Card”) with definition, uses, non-uses, trade-offs, traps, and mini-examples.
2) **Long-term retention**: the content is designed for practice with questions, evaluation, and periodic review (later: gamification + scheduling).

## How it’s used today (version 1: static Markdown)
 Topics live in `topics/` organized by categories (including a dedicated `aws/` section for AWS services).
 Each topic follows a common structure (see `templates/topic.template.md`).
 No duplicated explanations: navigate with links between concepts.
 Assessments (MCQ, open, problem-solving) live in `assessments/`.

## Philosophy: “Graph, not encyclopedia”
 `topics/`: topic cards (with subfolders by category, including `aws/`)
 `assessments/`: MCQ, open, and problem-solving question banks
 `templates/`: templates
 `prompts/`: strict prompts to generate and evaluate content
- `docs/`: vision, rules, taxonomy, authoring guide
 Assessments and sets (if any) live in `assessments/`.
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
