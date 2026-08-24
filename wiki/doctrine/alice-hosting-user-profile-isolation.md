---
title: alice-hosting-user-profile-isolation
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/alice-hosting-user-profile-isolation.md
updated: 2026-07-24
---

# Alice-as-User-Profile Isolation Model Evaluation

> Output of [EVOC-326](https://linear.app/lsctech/issue/EVOC-326/03-evaluate-alice-as-user-profile-isolation-model): evaluation of running Alice under a separate OS user account as a desktop automation isolation model.

This document evaluates the OS user-profile isolation model as a candidate mechanism for limiting blast radius from desktop automation. It is written after the privileged broker design ([EVOC-325](https://linear.app/lsctech/issue/EVOC-325/02-design-privileged-execution-broker-for-connect)) to avoid proposing elevation patterns that conflict with or bypass the broker. The recommendation feeds into the governance mapping ([EVOC-323](https://linear.app/lsctech/issue/EVOC-323/04-map-governance-and-audit-rules-across-hosting-models)) and v1 strategy selection ([EVOC-327](https://linear.app/lsctech/issue/EVOC-327/05-select-v1-always-on-alice-deployment-strategy)).

---

## 1. Model description and isolation boundaries

In the user-profile isolation model, Alice (the anchor runtime and its worker processes) runs under a **dedicated OS user account** (`alice` or similar) rather than the primary user's account. The primary user's session remains the owner of the interactive desktop environment; Alice operates as a background user with its own home directory, credential store, and filesystem namespace.

### Isolation boundaries provided

| Boundary | Mechanism | What is protected |
|---|---|---|
| Filesystem namespace | POSIX user/group permissions | Alice's writes are confined to `/home/alice` (or macOS equivalent `~alice`). Primary user's `~` is read-protected by default ownership rules. |
| Credential stores | Per-user keychain / secret-service | Alice cannot access the primary user's Keychain, browser credentials, SSH keys, or tokens without explicit cross-user grant. |
| Process namespace | UID-based process isolation | Alice cannot attach to, inspect, or signal the primary user's processes without ptrace privileges (disabled by default on modern macOS/Linux). |
| Network namespace (partial) | None by default | Both users share the same network stack. Port binding conflicts are possible; loopback access is shared. |
| Desktop session | Display server access controls | Alice cannot read the primary user's screen or inject input events without explicit X11/Wayland authorization or macOS Accessibility grants to Alice's UID. |
| Audio/camera/microphone | macOS TCC / Linux device permissions | macOS TCC controls access per-application (not per-user), so isolation at this layer is application-level only. |

### Isolation boundaries NOT provided

- **Shared hostname and network stack**: Both users see the same interfaces, routes, and loopback.
- **Shared CPU/memory pools**: No resource cgroup or QoS guarantee unless configured separately.
- **macOS TCC is application-scoped, not user-scoped**: Screen Recording, Accessibility, Microphone, and Camera grants in macOS TCC apply to specific application bundles, not to UIDs. An application run by `alice` that was granted Accessibility access has the same grant surface as the same binary run by the primary user. This significantly weakens the isolation claim for desktop automation specifically.
- **System keychain**: The system keychain (distinct from the user login keychain) is accessible to any sufficiently privileged process regardless of UID.

---

## 2. Filesystem interaction constraints

Running Alice as a separate user introduces non-trivial filesystem friction for a desktop assistant:

### 2.1 Primary user's files

Alice's primary function includes reading and manipulating files on behalf of the user. Under the isolation model:

- **Read access to user files requires explicit grant.** The primary user would need to set ACLs, use a shared group, or expose files through a controlled mount point for Alice to read `~/Documents`, `~/Desktop`, etc.
- **Write access requires explicit grant or a broker-mediated path.** Without ACLs, Alice cannot write to the primary user's home directory at all.
- **Home directory ACL management is user-hostile.** macOS and Linux both support ACLs, but there is no standard UI for granting another OS user read/write access to a subset of files. This creates an operational burden every time Alice needs access to a new directory.

### 2.2 Relationship to the privileged broker

The broker (EVOC-325) uses per-action `fs.protected-read` and `fs.protected-write` action classes to handle files outside Alice's declared sandbox. Under the user-profile model:

- Files in the **primary user's home directory become broker-mediated paths** rather than files Alice can directly access.
- The broker would need to run as the **primary user** (or as a user with ACL grants from the primary user) to read and write those files on Alice's behalf.
- This creates a **two-identity broker problem**: the broker must bridge Alice's UID and the primary user's UID, which means the broker process either runs as the primary user (losing separation) or requires a dedicated credential grant mechanism for cross-user file access.
- The broker design in EVOC-325 assumes the broker executes actions with **minimum-privilege credentials** from the calling context. A cross-UID file operation does not fit cleanly into this model: there is no standard "minimum-privilege cross-user file delegation" primitive on macOS or Linux without either shared group membership or a FUSE-based intermediary.

### 2.3 Application data directories

Many desktop applications (browsers, IDEs, note-taking apps) store their data under `~` of the active user. Alice, running as `alice`, would see an entirely separate set of application data directories from the primary user — different browser profiles, different SSH known_hosts, different application config. Desktop automation tasks that target the primary user's application state would fail or require explicit cross-user bridging for each application.

---

## 3. Relationship to and compatibility with the privileged broker

The privileged broker (EVOC-325) was designed with the assumption that Alice runs **in the same user session as the primary user**. The broker is the mechanism that gates elevation; Alice is unprivileged within that session.

The user-profile isolation model introduces a **second user boundary** that interacts with the broker in the following ways:

### 3.1 Broker placement

The broker must be accessible to Alice's UID to receive action requests. Options:

| Option | Trade-off |
|---|---|
| Broker runs as primary user, exposes a local socket | Alice (alice UID) submits requests over IPC; broker validates and executes as primary user. This preserves broker design but requires the broker to hold a socket owned by a different UID, with access granted to `alice` via group or explicit ACL. |
| Broker runs as a shared system user | Neutral UID; both Alice and primary user interact with it. Increases broker attack surface; the broker now has more access than either user alone. |
| Broker runs as `alice` user | Broker can only act within Alice's permissions. Cannot access primary user's files. Defeats the purpose of the isolation model. |

The first option is most compatible with the broker design, but it means the broker process (running as the primary user) is a **persistent privileged-access surface** that can act on behalf of the `alice` user. This is an inversion of the broker's intent: the broker was designed to limit Alice's ability to act with the primary user's authority, not to grant Alice a persistent channel to primary-user actions.

### 3.2 Audit log ownership and visibility

The broker design requires that audit logs be **user-inspectable** by the primary user (EVOC-325 §5.2). If the broker runs as the primary user but serves Alice's requests, log ownership is straightforward. If the broker runs as `alice`, the primary user's Task Manager UI would need cross-user log access — a non-trivial implementation requirement.

### 3.3 TCC and desktop automation

macOS TCC (Transparency, Consent, Control) governs Accessibility and Screen Recording access, which are the core mechanisms for desktop automation. TCC grants in macOS 10.15+ are:

- **Application-bundle-scoped**, not UID-scoped.
- **Not transferable across user sessions** in the same way: the `alice` user would need its own TCC grants, which require the user to be logged in to a GUI session or use `tccutil` with admin rights.
- In a **background user context** (no GUI session for `alice`), macOS does not present TCC prompts. The `alice` user would need pre-seeded TCC grants, which require admin authorization at setup time. This is a one-time but non-trivial setup burden.

---

## 4. Integration complexity assessment

| Concern | Severity | Notes |
|---|---|---|
| Cross-user file access for primary user's documents | High | Every directory Alice needs to read/write requires explicit ACL setup. No standard tooling or UI for this on macOS. |
| Broker cross-UID bridging | High | Broker placement options all have meaningful trade-offs; none maps cleanly to the broker's current design. |
| macOS TCC grants for background user | High | Desktop automation (Accessibility, Screen Recording) requires per-app TCC grants that cannot be prompted in a background user context. Admin setup required. |
| Application data isolation (browser profiles, SSH, etc.) | High | Alice's `~` is entirely separate from the primary user's. Desktop automation of primary-user applications requires cross-user data access or application re-configuration. |
| Setup and onboarding | High | Creating a secondary OS user, setting ACLs, pre-seeding TCC, and configuring broker IPC is a significant installation burden compared to running Alice in the primary user's session. |
| Credential store access | Medium | Primary user's login keychain is inaccessible to `alice` without explicit unlock. `credential.read` broker actions become cross-user operations. |
| Audit log accessibility | Low | Manageable if broker runs as primary user; requires cross-user read if broker runs as `alice`. |
| Platform portability | Medium | macOS TCC, Linux PAM/ACL, and Windows account model differ substantially. A cross-user isolation model would need platform-specific implementations on all three. |

**Overall complexity rating: High.** The model requires non-trivial setup, persistent operational overhead, and architectural changes to the broker that conflict with its current design constraints.

---

## 5. Pros and cons

### Pros

- **Strong filesystem isolation by default.** Alice cannot accidentally or maliciously read or write primary user files without explicit grant. This is the most robust filesystem boundary short of a full VM.
- **Separate credential namespace.** Alice's own credential store is isolated from the primary user's. If Alice's runtime is compromised, the attacker does not immediately have access to the primary user's secrets.
- **Process signal isolation.** Alice cannot kill, pause, or trace the primary user's processes.
- **Clear audit boundary.** Actions taken by the `alice` UID are OS-level attributable, separate from primary user activity in system logs.
- **Future-compatible with stronger sandboxing.** The user-profile boundary is the foundation for more aggressive isolation (e.g., separate login session, cgroup constraints) if needed.

### Cons

- **Breaks desktop automation for primary user's context.** Most desktop automation targets the primary user's active session, open applications, and data. The user-profile model makes this require explicit cross-user bridging for every access.
- **Conflicts with the broker's design model.** The broker was designed to gate elevated actions for Alice running in the primary session. Cross-UID bridging requires broker placement options that either weaken isolation or expand broker attack surface.
- **macOS TCC is not UID-scoped.** The headline isolation benefit (can't access the primary user's screen/input) does not hold on macOS for TCC-governed resources, which are application-scoped. The isolation claim is weaker than it appears on the primary target platform.
- **High setup complexity.** End-user setup of a secondary OS user with correct ACLs, TCC pre-seeding, and broker IPC is an onboarding barrier inappropriate for a v1 release.
- **Operational overhead.** ACLs and TCC grants need to be updated as Alice's scope changes. This is ongoing maintenance with no standard tooling on macOS.
- **No benefit within the broker model.** The broker already provides the primary safety property (Alice cannot execute elevated actions without explicit authorization). The user-profile model adds operational complexity without adding safety properties that the broker does not already provide in the primary-session model.

---

## 6. v1 Recommendation: **Defer**

**Decision: Defer the Alice-as-user-profile isolation model for v1.**

### Rationale

The user-profile isolation model provides genuine safety benefits in environments where Alice must not access the primary user's filesystem or application data at all. However, for EVOconnect's v1 use case — a desktop automation assistant that operates on behalf of the primary user — the model creates more friction than safety value.

The core reasons for deferral:

1. **The broker already provides the critical isolation.** The privileged execution broker (EVOC-325) gates all elevated actions behind ToolGrant validation, scope enforcement, and audit logging. Alice running in the primary session is already prevented from taking arbitrary elevated actions. The user-profile model does not add a meaningful safety layer on top of this.

2. **Desktop automation requires primary-user context.** Operating on the primary user's open applications, reading their files, and acting within their active session is the product's core function. The user-profile model makes every such operation a cross-user bridging problem.

3. **macOS TCC weakens the isolation claim on the primary target platform.** TCC's application-scoped (not UID-scoped) model means the most sensitive desktop automation permissions (Accessibility, Screen Recording) are not actually isolated by OS user account. The isolation benefit is substantially reduced on macOS.

4. **Integration complexity is high relative to v1 scope.** Cross-UID broker bridging, ACL management, TCC pre-seeding, and per-directory access grants are each individually non-trivial. Combined, they represent an implementation and onboarding burden that is out of scope for a v1 delivery.

5. **Does not conflict with privileged broker constraints.** This recommendation is to defer, not reject. The broker design does not preclude a future cross-UID model — it just means the broker would need extension work (cross-user IPC, UID-aware validation) before the model could be used safely. Deferring avoids designing elevation patterns that bypass the broker.

### Conditions for future adoption

The user-profile model becomes worth revisiting when:

- Alice is used in **multi-user or shared-machine contexts** where strict cross-user data isolation is a hard requirement.
- **macOS TCC evolves** to support UID-scoped grants (or an equivalent mechanism) for Accessibility and Screen Recording.
- The broker design is extended to support a **cross-UID delegation primitive** (e.g., the broker process acts as a shared intermediary with explicit grants to both UIDs).
- There is a demonstrated **threat model** that requires filesystem isolation stronger than the broker's per-action scope enforcement.

---

## Acceptance criteria coverage

- [x] **Model documented with isolation boundaries described** (Section 1)
- [x] **Pros and cons listed** (Section 5)
- [x] **Integration complexity assessed** (Section 4)
- [x] **Explicit recommendation made: Defer for v1** (Section 6)

## Related

^[source-materials/mirrors/doctrine/alice-hosting-user-profile-isolation.md]
