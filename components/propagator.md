## Propagator - Component Overview

The Propagator is Lucius’ **configuration authority and state anchor**. Conceptually, it plays the same role for Lucius that the Ben Ledger plays for Ben.

It exists to ensure that **configuration, intent, and component behavior are explicit, versioned, and recoverable**, without participating in the data hot path.

---

### Purpose

The Propagator serves as the **single source of truth** for Lucius configuration, including:

- Component enablement
- DSL artifacts (lyara, lstran, lthreat, lfin, assessor configs)
- Thresholds, bounds, and interpretation profiles
- Operational parameters required for deterministic behavior

It is hydrated exclusively through **benctl**, intentionally avoiding additional control tooling and preventing tool sprawl.

---

### What the Propagator Does

- Stores authoritative, versioned configuration for Lucius
- Applies configuration updates via explicit propagation events
- Records:
  - Successful hot swaps
  - Failed or rejected configuration changes
- Maintains snapshot state for recovery
- Rehydrates Lucius components after:
  - Crash
  - Restart
  - Partial system failure

The Propagator ensures that Lucius can always answer:
> “What configuration was in effect when this artifact was processed?”

---

### What the Propagator Does Not Do

- Participate in artifact processing
- Sit on the data hot path
- Make policy or risk decisions
- Interpret signals, scores, or outcomes
- Mutate artifacts
- Perform validation beyond structural correctness

The Propagator **does not reason**, it distributes.

---

### Authority Model

- The Propagator has **configuration authority only**
- It does not override runtime outcomes
- It does not infer intent
- It does not adjudicate conflicts

Runtime components consume propagated state but remain bounded by their own roles.

---

### Failure Philosophy

- Configuration state is snapshotted
- Snapshots are treated as recoverable checkpoints
- On failure:
  - Last-known-good configuration is restored
  - Components rehydrate deterministically
- Partial propagation is explicitly recorded

Configuration loss is treated as a critical failure mode and is designed against accordingly.

---

### Design Invariants

- No implicit configuration
- No silent drift
- No out-of-band mutation
- All changes are versioned
- All propagation is observable

The Propagator exists to make **configuration boring, durable, and trustworthy**, so the rest of Lucius can operate with confidence.