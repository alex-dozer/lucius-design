## Normalize - Component Overview

The Normalize component exists to make incoming artifacts **legible, consistent, and consumable** for downstream Lucius components.

It is intentionally simple.

Normalize is not an analysis stage, not a policy engine, and not an interpretation layer. Its sole responsibility is to ensure that artifacts conform to a **global, internal representation** that other components can reliably operate on.

---

### Purpose

Normalize provides a **canonical artifact schema** that downstream components depend on for correctness and determinism.

Without normalization, every component would need to defensively re-interpret inputs, increasing complexity, ambiguity, and error surface.

---

### What Normalize Does

- Converts raw artifacts into a standardized internal representation
- Ensures required structural fields are present
- Performs basic, non-semantic validation
- Performs necessary hashing as given by lthreat
- Establishes invariant guarantees required by:
  - Structural analysis
  - YARA analysis
  - Threat feed matching
  - Static analysis
  - Finalization

Normalize enforces *shape*, not meaning.

---

### What Normalize Does Not Do

- Make risk determinations
- Assign scores, weights, or confidence
- Apply intent
- Emit signals or risk hints
- Perform heuristics or pattern matching
- Attempt recovery or guessing

If Normalize cannot safely produce a valid internal representation, it does **not** attempt to fix or reinterpret the artifact.

---

### Failure Behavior

- Artifacts that cannot be normalized are:
  - Explicitly rejected
  - Recorded via the Notary
  - Annotated with failure context

There is no silent fallback.

Failure at this stage is treated as an **inability to reason**, not an error to be hidden.

---

### Authority Model

- Normalize has **no policy authority**
- It does not infer intent
- It does not influence routing decisions beyond pass/fail

Normalize is a **gate**, not a judge.

---

### Design Philosophy

Normalize exists to:
- Reduce downstream complexity
- Preserve determinism
- Eliminate implicit assumptions
- Make failures obvious and explainable

If an artifact cannot be normalized, Lucius explicitly records that fact and moves on.

Simple, strict, and boring, by design.