# Design Invariants

These invariants define the non-negotiable constraints under which Lucius is designed and evolved.  
They exist to limit ambiguity, prevent silent complexity creep, and preserve operator trust over time.

Lucius may grow in capability, but it must not grow in hidden authority.

---

## Authority & Control

### Intent is authoritative
Declared operator intent is the highest-order input to the system. All analysis, scoring, routing, and escalation occur **within** declared intent and never in place of it.

### No implicit authority
No component gains authority by convenience, position, or emergent behavior. Authority must be explicitly granted, scoped, and versioned.

Routing is not decision-making.

### Humans are part of the system
Lucius is not a closed loop. Human review, override, and judgment are first-class system states, not exceptional failures.

---

## Failure, Loss, and Truth

### Loss must be explicit
Dropped artifacts, rejected analysis, truncated data, and aborted stages must be recorded as outcomes. Silent loss is treated as a failure mode.

### Uncertainty must surface
Lucius may report low confidence, ambiguity, or insufficient signal. It must not fabricate certainty to simplify downstream handling.

### Rejection is still an outcome
Artifacts that are discarded or short-circuited still pass through finalization and are recorded with context and reasoning.

---

## Determinism & Drift

### No implicit drift
Behavior does not change without a version boundary. Rules, DSL semantics, scoring logic, thresholds, and routing criteria are versioned artifacts.

### Same inputs, same version, same outcome
Given identical inputs and configuration, Lucius behaves deterministically. Any non-determinism must be explicit and justified.

**The propagator may distribute state, but it may not introduce new semantics.**

---

## Throughput & Cost

### Expensive analysis is earned
High-cost operations (deep static analysis, emulation, detonation, heavy ML) are gated by prior signals. Nothing expensive runs “just in case.”

### The default path favors throughput
The common path through the system is optimized for volume and clarity, not maximal insight. Depth is opt-in, not ambient.

### Backpressure is a signal
Blocked channels, saturation, and queue pressure are observable system states and may influence scoring and escalation.

---

## DSL & Expressiveness

### DSLs are expressive but not Turing-complete
Lucius DSLs are intentionally constrained to preserve analyzability, predictability, and bounded execution.

### Rules describe intent, not programs
Rule definitions declare what is being looked for, not how to compute arbitrary logic.

### Scoring remains legible
Scoring mechanisms must be explainable to an operator without reverse engineering internal state or model internals.

---

## System Boundaries

### Lucius is not a detonation engine
Lucius may prepare, classify, and route artifacts, but execution in hostile environments belongs elsewhere.

### Lucius does not promise completeness
Coverage gaps are expected, documented, and visible. Claims of “catching everything” are explicitly rejected.

### Lucius does not encode best practices
Lucius provides mechanisms, not prescriptions. Policy lives with the operator.

---

## Observability & Explainability

### Every decision has a trace
All meaningful actions can be traced back to inputs, rules, versions, and declared intent.

### Metrics never replace reasoning
Quantitative signals inform decisions but never substitute for explicit justification.

---

## Maintainability

### Lucius must be explainable to its future maintainers
Designs that require tribal knowledge or oral tradition to understand are considered failures.