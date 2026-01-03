# lfin - Lucius Finalization & Decision Component

## Overview

`lfin` is the **finalization and decision component** of Lucius.  
It is responsible for taking accumulated signals, risk hints, statistics, and metadata produced by upstream components and **producing a coherent, explainable outcome** that can be:

- Stored in the Lucius ClickHouse silo
- Returned to Ben as a structured result
- Used to trigger bounded downstream actions

`lfin` does **not** detect threats.  
It **interprets** what has already been observed.

Its purpose is to turn many partial truths into a single, auditable decision surface.

---

## What lfin Does

- Consumes:
  - Signals (hard-coded enums emitted by upstream components)
  - Risk hints (bounded interpretive context enums)
  - Statistical outputs from `lstats`
  - Scores accumulated by YARA, structural analysis, and threat feeds
  - Operator-declared intent (via the propagator)

- Normalizes all inputs into:
  - Deterministic scores
  - Structured verdicts
  - Human-readable explanations

- Applies **intent-aware decision logic**:
  - What matters depends on what the operator declared
  - The same signals can produce different outcomes under different intent

- Coordinates **bounded actions**:
  - Quarantine
  - Escalation
  - Deep scrub requests
  - Ben-facing signals

- Produces a **final artifact record**:
  - Fully explainable
  - Replayable
  - Versioned

---

## What lfin Does *Not* Do

- It does **not** perform detection
- It does **not** invent facts
- It does **not** mutate upstream signals
- It does **not** run YARA, structural analysis, static analysis, or threat feeds
- It does **not** rely on stringly-typed tags for logic
- It does **not** infer intent
- It does **not** take irreversible action outside declared bounds

If something was not observed upstream, `lfin` cannot create it.

---

## Authority Model

`lfin` has **interpretive authority**, not observational authority.

- Facts come from upstream components
- Meaning comes from operator intent
- `lfin` is the only place where these two are reconciled

Authority boundaries:

- Signals are authoritative facts
- Risk hints are contextual modifiers
- Tags are non-authoritative metadata
- Intent is the highest authority

`lfin` may **decide**, but it may not **assume**.

---

## Goals

- Produce deterministic, explainable outcomes
- Preserve semantic meaning across the pipeline
- Prevent string-based coordination errors
- Centralize decision-making without centralizing detection
- Make outcomes legible to both humans and machines
- Enable Ben to reason about Lucius outputs reliably
- Allow policy evolution without rewriting detection logic

---

## Non-Goals

- Acting as a rule engine for detection
- Being a general-purpose policy language
- Supporting arbitrary scripting
- Encoding heuristics as truth
- Hiding uncertainty
- Optimizing for “one-click” simplicity
- Making finality irreversible by default

`lfin` is not designed to be clever.  
It is designed to be **correct and accountable**.

---

## Failure Philosophy

Failure in `lfin` is expected and first-class.

### Principles

- Failure is explicit, never silent
- Partial information is allowed; fabricated certainty is not
- Decisions degrade conservatively under uncertainty
- Outputs remain explainable even when incomplete

### On Upstream Failure

If upstream components fail or defer:

- `lfin` records missing context explicitly
- Decisions reflect reduced confidence
- No attempt is made to “fill in the gaps”

### On Internal Failure

If `lfin` encounters:
- Invalid intent
- Version mismatches
- Incompatible signal sets

It will:
- Emit a failure verdict
- Preserve all contributing inputs
- Avoid irreversible actions

### On Recovery

- Behavior is replayable via versioned inputs
- No hidden state is required for reconstruction
- Determinism is preserved across restarts

Failure is treated as **information**, not an exception.

---

## Summary

`lfin` is where Lucius becomes understandable.

It does not detect threats.  
It does not chase certainty.  
It does not replace human judgment.

It exists to ensure that **every outcome has a reason**,  
**every action has a boundary**,  
and **every decision can be explained**.