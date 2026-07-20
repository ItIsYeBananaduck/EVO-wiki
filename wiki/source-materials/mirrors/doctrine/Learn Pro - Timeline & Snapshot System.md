# Learn Pro - Timeline & Snapshot System

Each update creates an Approach Snapshot.

Snapshot contains:
- Name
- Created date
- Selected techniques
- Scope rules
- Source summary
- Internal model reference

Active snapshot influences Mesh.

“Forget” behavior:
- If previous snapshot exists → revert.
- If none → remove custom style entirely.

Timeline shows:
- Active
- Archived
- Source count