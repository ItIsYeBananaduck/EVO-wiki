---
title: EVOTRA-95-e2ee-trust-model-verification
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOTRA-95-e2ee-trust-model-verification.md"]
updated: 2026-07-24
---

# EVOTRA-95 — E2EE + Trust Model Adequacy Verification (EVOtraining Chat)

Date: 2026-04-19 (UTC)
Scope: `flutter_app` + `packages/core` chat runtime and Supabase chat schema/policies.

## Executive conclusion

**Conclusion: partially E2EE-capable, but trust model is not yet adequate for a strong E2EE claim.**

What is supported by code evidence:
- Chat payload encryption/decryption primitives exist and are applied before transport/storage.
- Supabase `chat_messages` schema stores encrypted payload fields (`encrypted_content`, `iv`) instead of plaintext.
- P2P transport carries encrypted envelopes over WebRTC DataChannel.

What blocks a full adequacy claim:
- Identity-to-key authenticity is weak (directory-based key lookup without user-verifiable fingerprints, signed key material, or pinning workflow).
- No evidence of end-to-end initialization wiring for `ChatSyncService` resolvers/transports in this repo snapshot.
- Key agreement model uses static identity keys and deterministic conversation key derivation without ratcheting/rotation semantics, so compromise blast radius appears broader than modern forward-secret messaging designs.

## Evidence-backed findings

## 1) Message confidentiality path

### 1.1 Encryption implementation
- `ChatCrypto` derives a shared secret via X25519 and derives a symmetric key via SHA-256 over `(conversationId || sharedSecret)`.
- Payload encryption uses AES-GCM with per-message random 12-byte nonce.
- Decryption verifies MAC via AES-GCM before plaintext is returned.

**Assessment:** Confidentiality/integrity primitives are present and modern enough for baseline encrypted transport/storage.

### 1.2 Local persistence
- Local chat store persists `ciphertext_b64` and `nonce_b64` only (not plaintext).

**Assessment:** Device-local store appears ciphertext-only at rest for chat payloads.

### 1.3 Runtime transport behavior
- `P2pChatService.sendMessage()` encrypts plaintext first and sends ciphertext+nonce envelope over P2P DataChannel.
- If P2P succeeds, current code still performs a best-effort Supabase backup send via `_syncService.sendMessage(...)`.
- If P2P is unavailable, it falls back to Supabase send.

**Assessment:** External storage path remains in the runtime design when fallback/backup is active; this is not pure P2P-only runtime.

## 2) Key distribution and identity binding

### 2.1 Server-side key directory model
- Supabase migration creates `chat_public_keys(user_id, public_key, ...)`.
- RLS policy allows authenticated users to read key rows; users can only write their own row.

**Assessment:** There is a server key directory with account-level write restrictions.

### 2.2 Binding strength
- The key directory binding is to Supabase auth identity (`auth.uid() = user_id`) at policy level.
- No evidence in current chat client flow of:
  - signed key records,
  - cross-signing,
  - fingerprint verification UI,
  - trust-on-first-use warning flow,
  - key transparency/audit proof checks.

**Assessment:** Identity binding is operational but not strongly authenticated against directory tampering or high-privilege service compromise. This weakens trust model adequacy for strict E2EE doctrine.

## 3) Can external services read message content?

## 3.1 Supabase database
- Schema stores encrypted payload fields, so plaintext content is not directly stored server-side when client encryption is used correctly.

**Readable by Supabase:** metadata (sender, recipient, timing, conversation identifiers), and ciphertext blobs.

## 3.2 Signaling service
- WebRTC signaling client sends/receives offer/answer/ICE over websocket endpoint with room/user identifiers.
- Signaling service necessarily sees connection metadata and SDP/ICE payloads.

**Readable by signaling service:** signaling metadata and session negotiation data, not chat plaintext payloads from DataChannel.

## 3.3 Caveat on trust claim
- Because key lookup is server-directory based and not strongly authenticated to users, a malicious/compromised directory authority could potentially influence key material and break effective E2EE guarantees (depending on attack path).

**Assessment:** “No external service can read content” cannot be claimed with high confidence under current trust model.

## 4) Material implementation uncertainty discovered

The repo snapshot contains unresolved wiring signals that prevent certainty about shipped behavior:
- `chatSyncService` is declared as a global `late final` in core, but no concrete initialization wiring was found in this snapshot.
- `P2pChatService` requires `peerPublicKeyResolver`; default path throws if resolver is missing.
- Chat UI constructs `P2pChatService` without explicit resolver injection (see `flutter_app/lib/features/chat/presentation/trainer_client_chat_screen.dart` lines 117-121: `P2pChatService` constructor call does not pass `peerPublicKeyResolver`).

This may indicate either:
1) missing initialization code outside inspected scope, or
2) incomplete runtime wiring in current branch.

Per issue requirement, this uncertainty is preserved rather than collapsed.

## Adequacy verdict by requested dimension

- **Key distribution adequacy:** **Partial / Not adequate for high-assurance E2EE.**
  - Works as a directory concept, but lacks robust authenticity controls.
- **Identity binding adequacy:** **Partial.**
  - Account-level DB policy binding exists, but user-verifiable cryptographic identity binding is insufficient.
- **External readability of message content:** **Conditionally limited, not fully excluded by trust model.**
  - Payload is encrypted at rest/in transit, but current trust model does not eliminate directory-authority/readability concerns.

## Recommended next hardening steps (for follow-on issues)

1. **Add key authenticity layer**
   - Signed key bundles, fingerprint display, TOFU warnings, and key-change verification prompts.
2. **Define explicit key-rotation and compromise semantics**
   - Include rotation events, old-key invalidation, and user-visible safety states.
3. **Move from static conversation key toward ratcheting/session keys**
   - Reduce blast radius if a long-term key is compromised.
4. **Produce runtime wiring proof artifact**
   - Explicit initialization path for `ChatSyncService` providers/transports and key publication/resolution.
5. **If doctrine requires “no external readable holder”**
   - Enforce P2P-only runtime path (no Supabase backup/fallback in live path) and keep server role strictly metadata/signaling-limited.

## Related
