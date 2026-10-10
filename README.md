# GhostGuard Voice Approval (prototype)

**Answer a pending GhostGuard `ask` with your voice — the recording becomes hash-bound evidence in the trail.**

An agent's action is recorded by GhostGuard with verdict `ask`. Instead
of clicking a button, the human records a short voice note stating the
decision. This tool appends a normal GhostGuard **decision entry**,
bound by SHA-256 to the ask's exact payload and to the recording's
bytes, and keeps the recording verbatim in an evidence folder that can
be re-verified later.

By **Ghost Developer Studio**. Standalone prototype (idea #12 from the
GhostGuard more-ideas list) — private, unpublished, nothing deployed.

## The honest identity statement — read this first

**Voice-match identity verification is NOT implemented.** This tool
does not transcribe the recording, does not check what was said, and
does not identify the speaker. In v1, voice is the **channel** and the
recording is the **evidence**; the `approver` name is self-asserted
and whatever identity exists comes from the human's own
device/session — **phone unlock is the identity anchor in the
MrGhosty path (see SPEC.md), the voice is not.** Design and roadmap:
[`SPEC.md`](SPEC.md).

## Quick start

```sh
# see what's waiting for a human
node src/cli.mjs pending --trail trail.jsonl --decisions decisions.jsonl

# decide one with a voice recording (any format — treated as opaque bytes)
node src/cli.mjs decide --trail trail.jsonl --decisions decisions.jsonl \
  --evidence evidence/ --action ask-1 --audio voice-note.wav \
  --decision approve --approver Ryan

# later: prove the stored recordings haven't been altered
node src/cli.mjs verify --decisions decisions.jsonl --evidence evidence/
```

## Library

```js
import { decideWithVoice, listPending, verifyEvidence } from "./src/voice-decide.mjs";

const entry = decideWithVoice({
  trailPath: "trail.jsonl",
  decisionsPath: "decisions.jsonl",
  evidenceDir: "evidence/",
  actionId: "ask-1",
  audioPath: "voice-note.wav",
  decision: "approve",        // or "deny"
  approver: "Ryan",
});
// entry.payload_sha256 — binds the decision to the ask's exact payload
// entry.audio_sha256   — binds it to this recording; entry.audio_file
//                        names the verbatim copy in the evidence folder
```

## Safety rules (enforced, tested)

- **Only `ask` / `hand_off` are decidable.** A voice decision can
  **never override a policy `deny`** (or an `allow`); deciding one
  errors and writes nothing.
- **Unknown `action_id` errors.** **Already-decided actions error** —
  decisions are append-only and cannot be re-decided or rewritten.
- **Payload binding:** the entry carries `payload_sha256` over the
  ask's canonical `action_payload`. Change the payload and the
  approval no longer matches it.
- **Evidence integrity:** `verify` re-hashes every stored recording;
  a recording edited, swapped, or deleted after the decision is
  reported by name (exit 1). Detection, not prevention — see SPEC.md.
- **The trail is never modified.** Decisions go to their own JSONL,
  one appended line each, in GhostGuard Cloud's decision shape
  (`entry_type: "decision"`, `decision: "approve"|"deny"`, …) plus
  `channel: "voice"` and the evidence fields.
- **The recording is never altered or transcribed** — copied byte for
  byte; its format is irrelevant to the tool.

## What's in the box

| File | What it is |
| --- | --- |
| `src/voice-decide.mjs` | Library: trail/decisions reading, `decideWithVoice`, `listPending`, `verifyEvidence`. |
| `src/cli.mjs` | CLI: `pending` / `decide` / `verify`. |
| `src/hash.mjs` | Canonical-JSON + SHA-256 helpers (payload binding, audio hashing). |
| `test/run-tests.mjs` | 14 checks — see REPORT.md. Fixture audio is synthetic (generated WAV header + silent samples, no speech, nobody's voice). |
| `SPEC.md` | MrGhosty integration path, identity roadmap, spoofing cautions, honest limits. |

Zero runtime dependencies; Node ≥ 18; ESM. The GhostGuard core package
is **read-only** to this prototype — it consumes GhostGuard's record
and decision shapes, it does not modify or require them at runtime
(trails arrive as JSONL files).

## Status

Prototype: local files only. No Cloud or MrGhosty wiring yet (the
integration path is specified in SPEC.md), no transcription, no
voice-match, no signing/hash-chaining of entries. `npm run verify`
runs the test suite.

## License

MIT — see the GhostGuard project. Copyright (c) 2026 Ghost Developer Studio.
