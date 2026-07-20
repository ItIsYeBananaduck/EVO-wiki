# Connect - Hive Architecture

Hive = coordination layer.

Responsibilities:
- Node discovery (WebRTC P2P)
- Fallback relay (Cloudflare if required)
- Capability detection
- Task distribution
- Encrypted internal communication

Rules:
- User never sees model-to-model chat.
- One leader node at a time.
- Nodes can enter/exit safely.
- Leader can be reassigned.

Hive ≠ Swarm.
Hive coordinates. Swarm computes.

#connect