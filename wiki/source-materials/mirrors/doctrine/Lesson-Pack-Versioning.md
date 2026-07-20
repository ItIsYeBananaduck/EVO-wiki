# Lesson Pack Versioning

## Default Behavior (Non-disruptive)
- Lesson Packs are versioned (v1, v2, v3…).
- If a student has started a lesson, they may finish the active version.
- New version becomes the default for students who have not started.

## Teacher Visibility
Teacher sees:
- who has v1 vs v2
- who started the lesson
- who completed it

## Student Awareness
Students are expected to be aware of changes through classroom instruction.
App may show a small banner:
“Lesson updated by your teacher.”

## Critical Updates (Enforced)
Teacher may mark an update as CRITICAL when:
- content error affects correctness
- objective changed
- assessment requirements changed

Critical update rules:
- requires resync before continuing
- preserves progress where possible (map attempts to new version)
- if mapping not possible, progress resets with a neutral message:
“Your teacher updated the lesson. Let’s continue with the new version.”

## Progress Preservation
When possible, migrate:
- completion flags for unchanged sections
- mastered items list
- spaced repetition queue