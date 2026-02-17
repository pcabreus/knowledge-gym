# Authoring guide

## Golden rule: no duplication
- If a topic exists for a term: link it.
- If it doesn’t exist: create a new topic.
- If someone uses another name for the same thing: create an alias in `topics/_aliases/`.

## Naming
- File: kebab-case, no accents, no spaces.
- Human title inside the file: can use normal capitalization.

## Structure
- Use `templates/topic.template.md` for topics.
- Use the appropriate assessment template for MCQ, open, or problem-solving (see `assessments/`).
- Keep sections and fill the applicable ones.
- Write in English. Technical terms in English (parentheses) the first time.

## Quality
- Avoid vague definitions.
- Include decision rules (when yes / when no).
- Include trade-offs (cost, complexity, performance, operability).
- Include 3+ trap questions with answers (for topics).

## Links
- Use relative links.
- Always link prerequisites and related topics.

## Assessments
All assessments (MCQ, open, problem-solving) must be placed in the appropriate folder under `assessments/`, not in `topics/`.
