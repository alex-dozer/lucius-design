## lop (Lucius Operation Plane)

The `lop` (Lucius Operation Plane) is the bounded execution layer responsible for carrying out **explicitly authorized actions** as the result of Lucius’ analysis pipeline.

It is an execution surface, not a decision-maker.

---

### What it does

- Executes **predefined, finite actions** approved by `lfin`
- Acts only on **explicit direction**, never inferred intent
- Serves as the bridge between Lucius analysis and downstream systems (e.g., `bux`, external tooling, export paths)
- Ensures actions are executed in a controlled, observable, and auditable manner
- Emits outcomes back into the Lucius audit and forensic record

---

### What it does not do

- Does not infer policy
- Does not perform scoring, classification, or interpretation
- Does not escalate actions autonomously
- Does not execute arbitrary or user-supplied code
- Does not modify intent or decision thresholds

---

### Authority boundaries

- `lop` has **no independent authority**
- It may only execute actions explicitly permitted by `lfin`
- All actions must be:
  - Predefined
  - Finite
  - Bounded in scope
- Any action outside this set is rejected by design

---

### Action model

- Actions are **declared ahead of time**, not dynamically constructed
- Examples include:
  - Exporting artifacts
  - Requesting deeper analysis
  - Triggering downstream workflows
  - Hydrating `bux` for forensic handling
- The existence of an action does not imply it will ever be executed; permission is granted per decision context

---

### Failure posture

- `lop` is **fail-soft**
- On execution failure:
  - The failure is recorded
  - Context is preserved
  - No retries are performed implicitly
- Failures do not cause escalation or alternate actions unless explicitly directed

---

### Role in the system

The `lop` exists to ensure that **action is deliberate, bounded, and accountable**.

Lucius may reason deeply, but it acts cautiously.

By separating interpretation (`lfin`) from execution (`lop`), Lucius preserves:
- Determinism
- Operator trust
- Clear accountability boundaries

`lop` is intentionally constrained so that power never outpaces intent.