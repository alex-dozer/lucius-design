# Lyara Subsystem Overview

The **Lyara subsystem** is Lucius’ bounded YARA execution engine. It is designed to provide **deterministic, intent-aware pattern matching** as part of a weighted, statistical sanitization pipeline; not as a general-purpose malware verdict engine.

Lyara exists to surface *signal*, not to assert truth.

---

## Purpose and Scope

Lyara is responsible for:
- Executing **bounded YARA-style pattern matching**
- Producing **weighted statistical signals**, not binary verdicts
- Feeding structured indicators into downstream components (Notary, Assessor, Finalize)
- Operating safely and deterministically under high throughput

Lyara is **not** intended to:
- Replace full YARA engines used in forensic labs
- Execute arbitrarily complex or unbounded rules
- Serve as a standalone malware classifier

---

## Core Dependencies

- **`lyara!` macro**
  - All rules are declared through the `lyara!` DSL
  - The macro enforces compile-time constraints and semantic validation

- **Structural Analysis**
  - Lyara relies on prior structural analysis to determine:
    - Artifact type (e.g., PDF, ELF, HTML)
    - Which rule classes are eligible to run
  - Structural signals gate rule execution before pattern matching begins

- **Propagator**
  - Lyara is hydrated exclusively through the propagator
  - Rule versions, thresholds, compiler selection, and intent are propagated explicitly
  - No local mutation or silent configuration drift is permitted

---

## Rule Execution Model

- **Rules do not run universally**
  - Lyara never runs “all rules on all artifacts”
  - Eligibility emerges from:
    - Structural analysis
    - Explicit rule metadata (e.g., declared file types)
    - Operator intent

- **Rule metadata may declare applicability**
  - File type constraints (e.g., `applies_to = pdf | docx`)
  - Analysis class hints (e.g., `active-content`, `packer-indicator`)
  - These declarations are advisory but enforced where possible

- **Pattern matching is selective**
  - Pattern execution is scoped to relevant artifacts
  - This prevents wasted compute and reduces false signal amplification

---

## Determinism and Throughput

Lyara is designed to be **strictly deterministic**:
- Identical inputs + identical configuration -> identical outputs
- Rule ordering, scoring, and actions are stable and reproducible

Throughput is a first-class concern:
- Rule constructs that historically cause locking, exponential behavior, or unbounded scans are discouraged or disallowed
- The DSL intentionally limits expressiveness where it would undermine system safety
- Complex analysis is deferred to later stages (e.g. static analysis) when justified

---

## Statistical Output (Not Verdicts)

Lyara does **not** return yes/no answers.

Instead, it:
- Produces **weighted statistical contributions**
- Emits tags, indicators, and confidence-adjusted signals
- Accumulates evidence rather than asserting maliciousness

Rationale:
- Binary answers collapse nuance
- Adversarial environments require gradient understanding
- Weighted systems allow downstream reasoning, aggregation, and override

---

## Relationship to Stats and Notary

Lyara **generates statistics**, but does not own global aggregation.

- Lyara:
  - Computes local weights and indicators
  - Emits structured statistical events

- Notary:
  - Accumulates statistics across pipeline stages
  - Applies declared statistical intent
  - Maintains the authoritative scoring ledger

This separation prevents Lyara from becoming a policy authority while preserving its analytical role.

---

## Assessor Interaction

Based on operator intent:
- Lyara may signal that an artifact meets criteria for deeper inspection
- The **Assessor**, not Lyara, decides whether static analysis is warranted
- Lyara provides evidence; it does not escalate unilaterally

This preserves bounded autonomy and clear authority boundaries.

---

## Compiler Strategy

Lyara supports **interchangeable compilers**:
- Different YARA backends may be selected based on environment or policy
- Compiler choice is explicit and versioned
- Compiler behavior is treated as part of the execution contract

This avoids coupling Lucius to a single YARA implementation.

---

## Failure and Recovery

- Lyara maintains no hidden state
- Upon catastrophic failure:
  - Configuration is rehydrated from the propagator
  - In-flight artifacts are recovered via upstream sliding window mechanisms
- Partial execution is never silently accepted

Failure is explicit and observable.

---

## Design Philosophy

Lyara exists because:
- Determinism matters more than cleverness
- Throughput matters more than expressiveness
- Understanding threats matters more than labeling them

Its constraints are intentional.

Lyara trades maximal power for **predictability, clarity, and trust** so operators can reason about outcomes instead of guessing why something happened.