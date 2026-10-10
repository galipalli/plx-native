---
name: evidence-first
description: >
  Decide what a claim is actually supported by, and how strong the evidence is. Use when a
  question is about implementation status, platform behavior, simulator or emulator results,
  or whether a result is verified versus planned. This skill forces a clear distinction
  between facts, assumptions, proposals, and unverified output before an answer is reported.
---

# evidence-first — claim only what the evidence supports

Use this skill before answering a question that might turn into a claim about implementation,
platform behavior, simulator fidelity, documentation status, or whether something is already
verified. This is a guard against plausible-but-unsupported statements.

## The rule

Every answer should say which of these categories the statement falls into:

- implemented and verified
- implemented but not verified here
- planned or proposed
- inferred or hypothesized
- simulator-only evidence
- emulator-only evidence
- physical-device evidence
- documentation-only reference
- unable to verify from the repo

A statement that does not attach a category is incomplete.

## What to do before delivering an answer

1. Check the repo first for implementation evidence.
2. Check whether the claim is a plan/specification rather than an implemented feature.
3. Check whether the result is host-only, simulator-only, or device-only.
4. Distinguish product reality from mock or sample behavior.
5. Name the missing evidence if the repo cannot prove the claim.
6. Prefer “I could not verify this” over a confident guess.

## Output pattern

```text
EVIDENCE: <implemented / planned / inferred / simulator / emulator / device / documentation only / unverified>
CLAIM: <the exact statement being made>
SUPPORT: <what in the repo or docs supports it>
LIMITATION: <what is not proven or what is missing>
```

## Examples

- “The repo contains a Tizen implementation” → needs a real Tizen directory, build, and verification.
- “This is a design proposal” → label as planned, not implemented.
- “This passes in the simulator” → label as simulator-only; do not call it device-verified.
- “This is a CLI example, not a validated command” → label as illustrative and unverified.
- “I could not verify this from the repo” → acceptable and preferred when no evidence exists.

## Consequences

A claim is not accepted just because it sounds right. A claim must either:

- be explicitly tied to evidence,
- be labeled as a proposal or an inference,
- or be rejected as unverified.

This skill is primarily a guardrail against fabricated certainty and stale or over-broad assertions.
