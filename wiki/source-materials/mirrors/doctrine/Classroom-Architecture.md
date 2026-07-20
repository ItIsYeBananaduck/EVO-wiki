# EVOlearn Classroom Architecture

## Entities
- Teacher Alice (TA): trained on lesson plans + teacher pedagogy ("how I teach")
- Student Alice (SA): per-student learning companion (Learn-only on school devices)

## Ownership
- TA belongs to the teacher identity/account.
- SA belongs to the student identity/account.

## Principle
TA provides teaching style + curriculum packaging.
SA provides individualized pacing + alternate explanations.

## Swarm
Per-student swarm (1–3 devices connected to that student’s Alice).
No classroom-wide compute pooling by default.
Workers are stateless for curriculum tasks.

## Domain Boundaries
- School device SA runs EVOlearn-only.
- Student phone may run full ecosystem, but Learn-only is enforced inside EVOlearn context.

## Substitute Mode (optional)
TA can drive class lesson delivery.
Substitute monitors classroom and operations.
TA does not manage discipline; TA manages instruction flow.