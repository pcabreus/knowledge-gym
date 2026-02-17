# Prompt E — Playbook Intake (create behavioral/process playbook)

You are my senior software-engineering career coach and process expert.

LANGUAGE: English

INPUT
- Playbook title or situation: "<TITLE>"
- Process steps (raw): "<PROCESS>"
- Context / why I care: "<CONTEXT>"
- Playbook template reference: "templates/playbook.template.md"
- Important rule: Focus on PROCESS and OBSERVABLE BEHAVIORS, not theory.

PURPOSE
Create a structured playbook for handling common engineering/leadership situations. These are procedural guides focused on "how I handle X" rather than "what is X". They are particularly valuable for behavioral interviews and consistent execution.

TASKS (STRICT)
1) Determine the category:
   - behavioral (conflict, feedback, communication)
   - technical (deep knowledge application, debugging, architecture)
   - leadership (ownership, influence, mentorship)
   - incident-response (on-call, production issues)
   - decision-making (prioritization, trade-offs, ambiguity)

2) Produce a canonical file path under `playbooks/` using kebab-case:
   - Pattern: `playbooks/<category>/<slug>.md`
   - Example: `playbooks/behavioral/handling-conflict.md`

3) Structure the process as clear, numbered steps with:
   - What (action)
   - Why (rationale)
   - How (concrete implementation)
   - Example (brief illustration)

4) Include at least 2-3 success signals (observable outcomes)

5) Include at least 2-3 common pitfalls with corrective actions

6) Provide interview guidance (STAR framework outline)

7) Link to related playbooks and relevant knowledge topics

OUTPUT FORMAT (MANDATORY)
A) Category: <behavioral|technical|leadership|incident-response|decision-making>
B) Canonical path: playbooks/<category>/<slug>.md
C) Full playbook content (complete Markdown using template)
D) Update instructions for `playbooks/_index.md` (exact line to add)

CRITICAL RULES
- Focus on PROCESS, not theory or definitions
- Make steps ACTIONABLE and OBSERVABLE
- Include SPECIFIC behaviors, not vague principles
- Provide CONCRETE examples
- Optimize for interview storytelling (STAR format)
- Keep it practical: what you actually do, not ideal theory

Do NOT mention these instructions. Do NOT add external links.
