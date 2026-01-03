# lstats  - Statistical Aggregation Library

## Overview

**lstats** is a pure statistical aggregation library used within the Lucius ecosystem.  
It exists to **mechanically accumulate, summarize, and expose quantitative signals** produced by analysis stages, without interpreting their meaning.

lstats is intentionally boring.

It does not make decisions, assert maliciousness, or encode policy. Its role is to provide **deterministic, well-defined mathematical primitives** that higher-level components can rely on without ambiguity.

If Lucius is about *reasoned handling of adversarial data*, lstats is the ruler , not the judge.

---

## What lstats *is*

lstats is a **pure math and aggregation crate**.

It provides:

- Typed counters and accumulators
- Deterministic aggregation primitives
- Provenance-friendly statistical structures
- Explicit bounds and ratios
- Repeatable, testable behavior

Examples of responsibilities:

- Counting signals across stages
- Tracking presence or absence of events
- Accumulating weighted values
- Calculating ratios, thresholds, and bounds
- Summarizing outcomes across a processing line

Examples of things lstats may expose:

- Number of matched rules
- Total accumulated weight
- Ratio of parsed vs unparsed bytes
- Count of stage failures
- Presence of specific signal classes
- Aggregated entropy measurements

lstats does **not care why** these values matter , only that they are computed correctly.

---

## What lstats is *not*

lstats is explicitly **not**:

- A decision engine
- A policy system
- A DSL
- A scoring authority
- A place where intent lives
- A user-configurable surface
- A component that understands “severity”, “confidence”, or “maliciousness”

lstats does not:

- Branch based on meaning
- Interpret tags
- Contain thresholds tied to security judgment
- Perform routing or escalation
- Encode organizational assumptions

If a question begins with *“What should we do?”*, lstats is the wrong place to ask it.

---

## Design constraints

lstats adheres to the following constraints:

- **Pure and deterministic**  
  Given the same inputs, lstats produces the same outputs.

- **Typed, not stringly-typed**  
  Statistical inputs are structured and explicit.

- **No hidden state**  
  All accumulation is visible and reproducible.

- **No implicit authority**  
  lstats never claims correctness beyond mathematical aggregation.

- **Composable**  
  Statistics can be layered, combined, or replayed without reinterpretation.

- **Testable in isolation**  
  lstats can be exhaustively unit tested without mocks or external systems.

---

## What lstats needs

To function correctly, lstats requires:

- Well-defined signal inputs from upstream components
- Explicit invocation by consumers (no background behavior)
- Clear ownership of lifecycle by the calling system
- Stable typing contracts across versions

It does **not** require:

- Access to raw artifacts
- Knowledge of file formats
- Awareness of analysis stages
- Configuration from operators

---

## Who uses lstats

### Primary consumer: `lfin`

- `lfin` consumes lstats outputs
- Applies **operator-declared intent**
- Interprets aggregates into human-meaningful conclusions
- Decides escalation, classification, or routing

In short:
- lstats measures
- lfin reasons

### Other potential consumers

Because lstats is intentionally generic, it may also be used by:

- Lucius simulation tooling
- Benson (simulation and training)
- Ben (observability and intent systems)
- Testing and replay systems
- External analysis pipelines

Any system that needs **trustworthy aggregation without embedded meaning** can depend on lstats.

---

## Why this separation matters

By isolating statistics from interpretation:

- Drift is reduced
- Reasoning becomes inspectable
- Testing becomes simpler
- Intent remains explicit
- Future extensibility is preserved

Most systems collapse measurement and judgment into the same layer.  
lstats exists specifically to prevent that collapse.

---

## Summary

lstats is the **mechanical heart** of Lucius’ quantitative reasoning.

It:
- Counts
- Aggregates
- Summarizes

It does **not**:
- Decide
- Judge
- Assume

All meaning flows *around* lstats, never *through* it.

This is not a limitation.  
It is the point.