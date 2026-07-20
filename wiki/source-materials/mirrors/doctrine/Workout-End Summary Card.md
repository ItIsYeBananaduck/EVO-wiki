# Workout-End Summary Card

## Concept
At workout completion, Alice generates a user-facing summary card.

## Rule / Mechanism
The summary card is generated only at workout end and must be derived from logged session data.

If the model is unavailable, a basic summary is shown and the enhanced summary may be deferred.

## Why It Exists
Workout end is a receptive moment for reflection without interrupting the session.

## Implications
- Works even if Alice is “cold”
- Enhanced summary can be produced later
- Supports a consistent UX pattern across apps

## Links
- [[Deferred Response Strategy]]
- [[Presence by Value]]
- [[Silent Failure Preference]]