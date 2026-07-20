# Ingestion Pipeline Reuse (Training → Learn)

## Existing capability
Training already converts:
- PDFs
- photos/scans
into structured Alice-usable data.

## EVOlearn extension
EVOlearn uses the same conversion primitives but compiles into:
- LessonPack schema
- MethodProfile
- Assessment items
- Unlock rules
- Safety constraints

## Rule
"Parse once, compile differently."
Training compilation targets nutrition/workout plans.
Learn compilation targets lesson units + practice + mastery gates.