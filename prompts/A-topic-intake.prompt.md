# Prompt A — Topic Intake (create/update topic without duplication)

You are my senior software-engineering knowledge curator.

LANGUAGE: English

INPUT
- New term (raw): "<TERM>"
- Context / why I care now: "<CONTEXT>"
- Repo taxonomy reference: "docs/04-taxonomy.md"
- Authoring rules: "docs/02-authoring-guide.md" and "docs/03-graph-and-links.md"
- Important rule: NEVER duplicate existing explanations. Prefer links.

TASKS (STRICT)
1) Determine if this term is:
   a) a new canonical topic
   b) an alias of an existing canonical topic
   c) a subtopic that should be merged into an existing topic (explain why)
2) Produce a canonical file path under `topics/` using taxonomy (kebab-case filename).
3) Output a complete topic Markdown file using `templates/topic.template.md`.
4) Include prerequisites and related topics as relative links. If a prerequisite topic does not exist yet, list it anyway (as a TODO).
5) Provide at least 3 trap questions with answers.
6) If alias is needed, also output an alias file content for `topics/_aliases/<alias>.md` that points to the canonical file.

OUTPUT FORMAT (MANDATORY)
A) Decision (canonical vs alias vs merge) + reasoning (max 8 lines)
B) Canonical path: <path>
C) Full content for canonical file (complete Markdown)
D) (If needed) Full content for alias file (complete Markdown)
E) Update instructions for `topics/_index.md` (exact lines to add)

Do NOT mention these instructions. Do NOT add external links.
