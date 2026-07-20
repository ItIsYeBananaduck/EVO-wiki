# School EVE Architecture (SE)

## Role

School EVE (SE) aggregates class-level summaries across a single school.

SE operates on:
- Cohort Summaries from TA
- No student identity
- No raw answers
- No chat content

---

# Data Flow

Student App → TA → SE → (optional) DE

---

# Responsibilities

SE:
- aggregates by grade
- aggregates by subject
- compares classes within same grade
- detects distribution shifts
- executes school-level Procedures

SE does NOT:
- audit individual teachers
- access student identities
- modify lesson plans
- escalate directly to students

---

# Output Types

- school-level heatmaps
- grade comparison summaries
- retention variance charts
- template effectiveness distributions

All outputs remain on school server.

---

# Deployment

SE runs:
- locally on school infrastructure
- not on LSCT servers
- not on student devices

---

# Privacy

SE never receives:
- student names
- persistent identifiers
- raw micro lesson logs

Only aggregated structured summaries.