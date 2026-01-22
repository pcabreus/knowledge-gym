---
id: versioning-apis-events-mcq-1
topic: ../topics/system-design/versioning-apis-and-events.md
kind: mcq
difficulty: medium
timebox_minutes: 4
created_at: 2026-01-22
updated_at: 2026-01-22
---

# API and event versioning

## Prompt
Which change requires a new version rather than a backward-compatible update?

## Answer format (what “good” looks like)
- Identify a breaking change
- Mention backward compatibility (backward compatibility)

## Answer key / Rubric
### Correct answer (for MCQ)
- Correct: D
- Why correct: Removing or renaming a required field is a breaking change that requires a new version or a deprecation plan.
- Why A is wrong: Adding an optional field is backward-compatible.
- Why B is wrong: Adding a new enum value can be backward-compatible if consumers are tolerant.
- Why C is wrong: Adding a new endpoint is additive and non-breaking.
- Why D is wrong: N/A

A) Add an optional field with a default.
B) Add a new enum value while keeping existing ones.
C) Add a new endpoint while keeping old ones.
D) Remove a required field from the payload.

## Common mistakes
- Changing semantics without a version bump.
- Assuming all enum changes are safe without consumer tolerance.
