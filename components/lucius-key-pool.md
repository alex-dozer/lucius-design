## lucius_key_pool

The `lucius_key_pool` is the stateful coordination component responsible for tracking which artifacts are currently eligible for processing within Lucius.

It functions as a **key registry and sliding window manager**, not as an authority on policy, risk, or outcomes.

---

### What it does

- Maintains a repository of artifact keys representing objects stored in backing storage (e.g. MinIO)
- Tracks **active keys** currently in-flight through the Lucius pipeline
- Maintains a **sliding window** of work in coordination with the `teller`
- Snapshots its state to allow recovery after failure
- Differentiates keys by domain:
  - Lucius keys (light scrub path)
  - Luxamen keys (deep scrub / detonation path)
- Provides a stable mechanism to resume work after interruption without reprocessing or loss of context

---

### What it does not do

- Does not make policy decisions
- Does not score, enrich, or classify artifacts
- Does not infer intent
- Does not decide routing beyond domain delineation (Lucius vs Luxamen)
- Does not participate in analysis or final decisions

---

### Authority boundaries

- `lucius_key_pool` is **authoritative only over key state**, not meaning
- Its authority is limited to:
  - Whether a key is active
  - Whether a key belongs to Lucius or Luxamen
  - Whether a key is eligible to be handed to the pipeline
- All interpretation of artifacts happens downstream

---

### Failure posture

- Operates with snapshot-based recovery
- On restart, rehydrates its last known sliding window
- Prevents silent key loss or duplicate processing
- Treats loss of key state as a critical failure that must be surfaced, not masked

---

### Role in the system

The `lucius_key_pool` exists to ensure **continuity, safety, and determinism** in artifact processing.  
It allows Lucius to fail, restart, and resume without ambiguity about what was being processed and why.

It is intentionally boring... and that is by design.