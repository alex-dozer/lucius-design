# lthreat  - Threat Feed Resolution Component

## Overview

**lthreat** is a narrowly scoped component responsible for **hash-based threat intelligence lookups**.

Its sole purpose is to take **pre-computed artifact hashes** and resolve them against **operator-configured threat feeds**, returning structured match results that can be reasoned about elsewhere in the system.

lthreat does not analyze content.  
It does not infer behavior.  
It does not decide maliciousness.

It answers one question, precisely and repeatably:

> “Does this artifact hash appear in any declared threat feed, and under what conditions?”

---

## What lthreat *is*

lthreat is a **deterministic lookup engine**.

It provides:

- Hash resolution against external or internal feeds
- Typed match results
- Feed provenance and confidence metadata
- Explicit failure and timeout semantics
- Replayable, auditable outcomes

lthreat operates only on **already-derived hashes** (e.g. SHA-256, SHA-1, MD5 if explicitly enabled).

It does not compute hashes itself.

---

## What lthreat is *not*

lthreat is explicitly **not**:

- A reputation engine
- A behavioral classifier
- A scoring authority
- A network crawler
- A heuristic system
- A place for fuzzy matching
- A substitute for analysis

lthreat does not:
- Interpret what a match “means”
- Collapse multiple feeds into a single truth
- Normalize trust across providers
- Hide feed disagreement
- Perform enrichment beyond reporting what the feed says

If a feed is wrong, stale, or contradictory, lthreat reports that fact , it does not correct it.

---

## Design constraints

lthreat adheres to strict constraints to preserve trust:

- **Exact-match only**  
  No similarity, no heuristics, no inference.

- **Feed-scoped truth**  
  A match is always tied to a specific feed.

- **No implicit authority**  
  Multiple feeds may disagree; all results are preserved.

- **Explicit failure**  
  Timeouts, errors, and unreachable feeds are recorded.

- **Deterministic behavior**  
  Given the same hashes and feeds, results are repeatable.

- **No side effects**  
  lthreat does not mutate state or trigger actions.

---

## Inputs

lthreat consumes:

- One or more artifact hashes
- Explicitly declared hash algorithms
- Operator-configured threat feeds
- Feed-specific configuration (timeouts, credentials, scope)

All inputs are hydrated via the **propagator**.

---

## Outputs

lthreat produces structured results containing:

- Hash algorithm used
- Feed identifier
- Match / no-match result
- Feed-provided metadata (if any)
- Timestamp of lookup
- Error or timeout state (if applicable)

No aggregation or scoring occurs in lthreat.

---

## Failure philosophy

Threat feeds are unreliable by nature.

lthreat treats failure as expected:

- Feed unavailable → recorded, not hidden
- Partial results → preserved
- Conflicting matches → reported as-is
- Slow feeds → bounded by configuration

Failure to resolve is not treated as absence of threat, it is treated as **unknown**.

---

## Who uses lthreat

### Primary consumer: `lfin`

- lfin interprets feed matches in the context of:
  - Operator intent
  - Other analytical signals
  - Accumulated statistics
- lfin decides whether a match escalates, annotates, or is ignored

### Secondary consumers

- Forensic review pipelines
- Offline analysis and replay
- Simulation and testing (e.g. Benson)

---

## Why lthreat is intentionally small

Threat feeds are volatile, political, inconsistent, and often wrong.

By keeping lthreat minimal:

- Feed bias is exposed, not absorbed
- Trust boundaries remain clear
- Operators retain judgment
- The system avoids cargo-cult security logic

lthreat answers *“What do feeds claim?”*  
It never answers *“What should we believe?”*

---

## Summary

lthreat is a **precision instrument**.

It:
- Resolves hashes
- Reports matches
- Preserves disagreement
- Records failure

It does **not**:
- Decide
- Score
- Infer
- Normalize truth

In Lucius, threat feeds are **inputs**, not authorities, and lthreat enforces that boundary.