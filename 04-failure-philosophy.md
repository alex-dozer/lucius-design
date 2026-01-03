## Failure Philosophy

### Failure is expected, not exceptional

Lucius operates in adversarial, high-entropy environments. Failure is not treated as an anomaly to be hidden or smoothed over, but as a normal operational condition that the system is explicitly designed to tolerate.

The system assumes that components will crash, inputs will be malformed, dependencies will fail, and partial state will exist. Design choices are made accordingly.

---

### Intent remains first-class under failure

Declared operator intent does not disappear during failure conditions.

When failures occur, Lucius continues to evaluate behavior relative to declared intent rather than attempting to “do something reasonable” on its own. No new authority is inferred simply because the system is under stress.

Failure does not grant Lucius additional latitude.

---

### Failure modes are explicit and observable

Lucius does not fail silently.

When an artifact cannot be processed, routed, analyzed, or finalized, that outcome is explicitly recorded with context, timestamps, and reason codes. Rejection is treated as a valid terminal outcome and is still finalized and persisted.

Artifacts are never silently dropped.

---

### Fail closed by default for safety-critical actions

Lucius fails closed by default for actions that could materially affect safety, trust, or downstream impact. This includes irreversible transformations, routing to detonation environments, or release of sanitized output.

Lucius may fail open only for telemetry or recording paths, and only when explicitly configured to do so.

This distinction is deliberate and explicit per stage.

---

### Bounded autonomy applies during failure

Lucius retains bounded autonomy during failure conditions, but that autonomy does not expand.

The system may continue to:
- Record state
- Preserve artifacts
- Accumulate partial context
- Surface uncertainty

The system may not:
- Invent confidence
- Perform irreversible actions outside declared bounds
- Mask or compress uncertainty for the sake of progress

---

### Authority is never inferred

No component assumes authority simply because another component failed.

Routing, escalation, rejection, and downstream actions always require explicit authorization through declared intent and versioned configuration. Failure does not create implicit permissions.

---

### Recovery is anchored to explicit state

Recovery is anchored to explicit, durable state; not memory.

Lucius snapshots and persists:
- The propagator state (versioned configuration, rules, thresholds)
- The lucius_key_pool
- A bounded sliding window of in-flight artifacts

This allows Lucius to resume processing from a known point without reprocessing finalized artifacts or losing forensic continuity.

---

### Data loss is mitigated, not denied

Lucius does not claim to eliminate data loss.

Instead, it mitigates loss by maintaining a bounded window of in-progress work that can be replayed on restart. This window includes artifact identity, last completed stage, attempt counts, and failure context.

Recovery is achieved through replay of a bounded in-flight window, not through unbounded workflow state.

---

## Processing is at-least-once, not exactly-once

Lucius assumes at-least-once processing semantics under failure.

Idempotence is achieved through stable artifact identities, deterministic stage behavior, and guarded finalization. Duplicate work is tolerated; duplicate outcomes are not.

“Exactly-once” guarantees are not assumed.

---

### Partial knowledge is labeled, not hidden

When analysis stages cannot run or complete, artifacts are explicitly marked as incomplete.

Downstream stages must respect incomplete context and may not silently assume missing information. Uncertainty is surfaced, preserved, and recorded.

---

### Backpressure is a signal, not an incident

Bounded channels and stalled pipelines are treated as legitimate system signals.

When Lucius experiences backpressure, it prioritizes correctness and traceability over throughput. Throughput degradation is explicitly recorded as an operational event.

---

### Failure surfaces design weakness

Failures are treated as feedback.

Any failure mode that requires manual intervention, ad-hoc state mutation, or undocumented recovery procedures is considered a design defect. Lucius treats recovery paths as part of the system design surface, not as operational afterthoughts.