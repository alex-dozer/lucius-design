# lstatic - Lucius Static Analysis Component

## Purpose

`lstatic` provides **bounded, deterministic static analysis** within Lucius. It does limited disassembly.

Its role is not to fully understand programs, complete decompile arbitrary formats, or act as a malware research platform. Its purpose is to **extract meaningful, explainable signals** from artifacts that justify deeper inspection, escalation, or forensic retention while remaining safe, predictable, and operator-controlled.

`lstatic` exists to answer questions like:

- *What capabilities does this artifact appear to expose?*
- *Does this artifact structurally or semantically conflict with declared intent?*
- *Are there strong indicators that justify deeper, more expensive analysis?*

It does **not** attempt to answer *“Is this definitively malicious?”*

---

## Static Analysis Tier

Lucius deliberately operates `lstatic` at:

### **Tier 1 - Shallow Semantic Inspection**

This tier extracts **observable signals** without building a compiler, interpreter, or execution environment.

It avoids:
- Full decompilation
- Control flow graph construction
- Symbolic execution
- Dynamic or emulated runtime behavior

This tier is chosen because it offers a **high signal-to-cost ratio** and preserves determinism.

---

## What `lstatic` Does

`lstatic` performs **feature extraction**, not interpretation.

Depending on artifact type and operator intent, this may include:

### Binary Artifacts
- Import table inspection
- Export table inspection
- String table extraction
- Section entropy analysis
- Suspicious API presence (e.g. memory injection primitives)
- Opcode pattern presence (without CFG construction)
- Hardcoded URLs, IPs, or command strings

### Scripted / Interpreted Artifacts
- Token-level keyword inspection
- Dangerous primitive detection (e.g. `eval`, `pickle.loads`)
- Suspicious library usage
- Embedded payload detection
- Hardcoded network endpoints

### General Signals
- Capability inference (networking, execution, persistence)
- Structural inconsistencies
- Known-bad indicator matches (via imported datasets)
- Confidence-weighted anomaly signals

All outputs are **facts**, not conclusions.

---

## What `lstatic` Explicitly Does *Not* Do

`lstatic` does **not**:

- Fully Decompile or reconstruct source code
- Build control flow graphs
- Perform taint analysis
- Execute artifacts
- Emulate runtime behavior
- Attempt language completeness
- Claim correctness or exhaustiveness
- Assert maliciousness as an absolute fact

These are **intentional exclusions**, not missing features.

Deeper analysis belongs to **Luxamen** or external systems designed for detonation, sandboxing, or research.

---

## Determinism and Safety

`lstatic` is deterministic by design.

Given the same artifact, configuration, and version:
- The same signals are produced
- The same scores are emitted
- The same routing decisions occur

There is:
- No adaptive learning
- No self-modifying logic
- No runtime code execution

This ensures:
- Reproducibility
- Auditability
- Predictable failure modes

---

## Data Model

Static analysis outputs are normalized into **derived signals**, not preserved program structure.

Key properties:
- Values are treated as facts, not events
- No temporal reasoning
- No relational joins
- No historical aggregation

This enables:
- Fast lookup
- Simple scoring
- Clear operator reasoning
- Minimal dependency surface

Forensic truth and historical context are handled **downstream**, not within `lstatic`.

---

## Routing and Escalation

`lstatic` does not make final decisions.

It may:
- Contribute weighted signals
- Attach capability tags
- Recommend escalation

Escalation (e.g. to Luxamen) occurs **only** when:
- Declared intent allows it
- Thresholds are met
- The assessor authorizes it

Artifacts denied deeper analysis continue through the Lucius pipeline with full context preserved.

---

## Configuration and Intent

`lstatic` behavior is governed by operator-declared intent.

Operators specify:
- Which signals matter
- Which artifact types are eligible
- What thresholds justify escalation
- What uncertainty is tolerable

Defaults are conservative, explicit, and visible.

There is no hidden policy.

---

## Failure Philosophy

Failure in `lstatic` is explicit.

Failures may include:
- Unsupported formats
- Partial extraction
- Parser limits
- Resource exhaustion

In all cases:
- The failure is recorded
- Context is preserved
- Downstream components are informed

There is no silent drop and no fabricated certainty.

---

## Design Constraints (Non-Negotiable)

`lstatic` is constrained by design to ensure it remains:

- Safe to operate continuously
- Predictable under load
- Understandable by operators
- Auditable in hindsight

These constraints exist to prevent:
- Accidental detonation engines
- Undebuggable heuristics
- Hidden execution paths
- Operator distrust

---

## Summary

`lstatic` is a **signal extractor**, not a judge.

It exists to surface meaningful evidence, not to replace human reasoning or downstream systems. By intentionally limiting scope, Lucius ensures static analysis remains a **tool**, not an opaque authority.

This restraint is a feature, not a limitation.