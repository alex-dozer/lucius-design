# Lstran - Structural Analysis

## Purpose

Lstran is the structural analysis component of Lucius.

Its responsibility is to establish **what an artifact is**, **what it claims to be**, and **how it is structured**, without executing, emulating, or behaviorally interpreting the artifact.

Structural analysis exists to surface *cheap truth*:
- What kind of data this is
- Whether that claim is credible
- What it contains at a high level
- How risky or expensive deeper analysis would be
- Which downstream analyses are applicable

Lstran does **not** decide maliciousness.  
It produces deterministic descriptors that inform later stages.

---

## Design Goals

Lstran is designed to be:

- **Deterministic**  
  Identical input bytes produce identical structural output.

- **Bounded**  
  All parsing, recursion, and inspection are constrained by explicit limits.

- **Non-executing**  
  Artifacts are never run, interpreted, or emulated.

- **Explainable**  
  Outputs are explicit facts, counts, flags, and reasons. Never opaque judgments.

- **Cheap**  
  Structural analysis must remain significantly less expensive than static or dynamic analysis.

---

## What Structural Analysis Is (and Is Not)

### Lstran *does*:
- Identify artifact type using observed bytes and structure
- Validate basic structural sanity
- Enumerate containers, sections, streams, or objects
- Detect mismatches between claims and reality
- Emit descriptors, flags, and hints for downstream routing

### Lstran *does not*:
- Execute code or scripts
- Emulate behavior
- Assert intent or maliciousness
- Perform deep semantic interpretation
- Replace YARA or static analysis

If a question sounds like *“what does this do?”*, it does not belong in Lstran.

---

## Methodology

Structural analysis proceeds through a bounded, staged process:

### 1. Identification

Establish what the artifact *appears* to be based on:
- Magic bytes
- Header signatures
- Encapsulation formats
- Encoding characteristics

Outputs:
- `claimed_type` (extension, mime, external metadata)
- `observed_type` (derived from bytes and structure)
- `confidence` (high / medium / low, with reasons)

---

### 2. Structural Validation

Perform minimal parsing sufficient to validate structural claims.

Examples:
- ZIP: central directory sanity, entry enumeration
- PDF: header, object count, stream presence
- PE: section table integrity, import table presence
- OOXML: relationship manifests, macro storage detection

Parsing is:
- Best-effort
- Fail-soft
- Explicitly bounded

Failures are recorded, not hidden.

> Fail-soft: A failure mode where partial results are preserved and uncertainty is made explicit, rather than aborting or concealing error.

---

### 3. Structure Extraction

Enumerate high-level structural properties, such as:
- Container nesting chains
- Section or stream counts
- Embedded object presence
- Compression methods and ratios
- External references (declared, not fetched)

No recursive or unbounded traversal is permitted.

---

### 4. Structural Heuristics (Non-authoritative)

Apply cheap, named heuristics that indicate *structural abnormality*, not intent.

Examples:
- Extension vs magic mismatch
- Extreme compression ratios
- Excessive nesting depth
- Unexpected executable sections
- High-entropy regions in structurally sensitive areas
- Malformed-but-parseable layouts

These produce **flags**, not verdicts.

---

## Output: Structural Descriptor

Lstran emits a structured descriptor object containing:

- Artifact identity
- Structural facts
- Counts and metrics
- Named structural flags
- Parsing failures (if any)
- Routing hints for downstream stages

Example fields (illustrative):

- `claimed_type`
- `observed_type`
- `container_chain`
- `size_metrics`
- `layout_metrics`
- `structural_flags`
- `yara_class_hint`
- `static_eligibility_hint`
- `safe_to_parse_further`

These descriptors are consumed by Notary and downstream components.

---

## Interaction With Other Components

### Notary
Lstran submits descriptors to the Notary, which:
- Records outputs
- Applies operator-declared weighting
- Accumulates context across stages

Lstran does not score artifacts directly.

---

### Lyara
Lstran emits **YARA class hints** to prevent wasteful rule execution.

Examples:
- `Pdf`
- `Pe`
- `Office`
- `Archive`
- `ScriptText`
- `WebContent`

Lyara selects rule families based on these hints.

---

### Assessor
Structural flags and metrics may inform whether an artifact is *eligible* for static analysis.

Lstran does not escalate directly; it provides evidence.

---

## Constraints and Guardrails

To preserve determinism and throughput, Lstran enforces:

- Maximum bytes inspected per artifact
- Maximum container recursion depth
- Maximum extracted metadata size
- Per-parser timeouts
- No external resolution (no network access)
- No unbounded decompression
- No speculative recovery loops

Violation of bounds results in explicit failure states, not retries.

---

## Failure Semantics

Structural analysis failures are expected and recorded.

Failures:
- Are explicitly logged
- Include reason and location
- Still produce a descriptor
- Never silently drop artifacts

Malformed or unsafe artifacts may be marked as unsuitable for deeper analysis, subject to operator intent.

---

## Design Philosophy

Lstran exists to answer four questions cheaply and reliably:

1. What is this artifact?
2. Is it lying about what it is?
3. What does it structurally contain?
4. What analysis paths even make sense next?

It intentionally avoids answering:
> “Is this malicious?”

Structural truth precedes behavioral judgment.

---

## Summary

Lstran establishes the **structural truth** of artifacts entering Lucius.

By emitting bounded, deterministic descriptors, it enables:
- Targeted YARA execution
- Informed static analysis decisions
- Reduced wasted computation
- Clear forensic reasoning

Structural analysis is the foundation that allows Lucius to scale without sacrificing clarity, intent, or trust.