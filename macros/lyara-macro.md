# Lucius – lyara! (Lucius YARA Macro)

## Purpose

`lyara!` is the **bounded pattern-matching DSL** used by Lucius Light Scrub for
deterministic, auditable artifact inspection. It is inspired by YARA, but
intentionally constrained to operate safely inside an authoritative sanitization
boundary.

This document defines:
- What the `lyara!` macro **is**
- What it **can express**
- What it **explicitly forbids**
- Why those constraints exist

This is a **design document**, not an implementation guide.

---

## Design Goals

- Deterministic execution
- Bounded time and memory
- Compile-time validation
- Explicit semantics
- Operator trust and auditability

`lyara!` exists to **surface signal**, not to perform unbounded research or
general-purpose computation.

---

## Non-Goals

`lyara!` is **not**:

- A full YARA replacement
- A scripting language
- A Turing-complete rule engine
- A place for environment introspection
- A runtime-mutable system

These exclusions are intentional.

---

## High-Level Structure

```rust
lyara! {
    meta { ... }
    actions { ... }
    rules { ... }
    conditions { ... }
}
```

Each block has **strict responsibilities** and **no hidden coupling**.

---

## Block Semantics

### `meta`

The `meta` block defines **human and system context**.

Allowed fields:
- `name`
- `author`
- `source`
- `version`
- `severity`
- `base_weight`
- optional handling hints (e.g. `forensics = true`)

Constraints:
- No logic
- No conditionals
- No runtime access

Purpose:
- Attribution
- Versioning
- Stable scoring inputs

---

### `actions`

The `actions` block declares **named outcomes**.

Examples:
- `quarantine`
- `extract_strings`
- `request_deep_scrub`
- `notify_soc`

Constraints:
- Declaration only
- No parameters
- No side effects at declaration time

Purpose:
- Provide a shared action vocabulary
- Allow reuse across rules and conditions
- Enable compile-time validation

---

### `rules`

The `rules` block defines **pure match primitives**.

Supported rule types include:
- `bytes`
- `ascii`
- `hash`
- `entropy`
- `filesize`
- `applies_to`

Common constraints:
- No recursion
- No rule-to-rule mutation
- No dynamic inclusion
- Bounded matching only

Each rule produces a **boolean or bounded numeric result**.

---

### `conditions`

The `conditions` block defines **scoring and decision logic**.

Supported constructs:
- `when`
- `not`
- `all(...)`
- `any(...)`
- `count N..M`
- comparative operators (`>=`, `<=`, etc.)

Side effects allowed:
- score accumulation
- tagging
- action enactment

Constraints:
- No loops
- No mutation outside score/tags
- Deterministic ordering

---

## Scoring Model

- Scores are **explicitly additive**
- No hidden weighting
- No implicit normalization
- Thresholds are operator-defined

This allows:
- Transparent reasoning
- Replayability
- Cross-version comparison

---

## Determinism & Safety Guarantees

`lyara!` guarantees:
- Bounded execution time
- Bounded memory use
- No side effects outside declared actions
- Compile-time rejection of unsafe constructs

Explicitly forbidden:
- Unbounded regex repetition
- Lookbehind
- External file access
- Environment access
- Rule imports

---

## Relationship to Lucius Pipeline

`lyara!`:
- Is invoked after structural normalization
- May feed the Assessor
- Never directly triggers static analysis
- Produces inputs for Notary and Finalize

It is **not** a control plane.

---

## Extensibility

`lyara!` is designed to evolve:
- New rule primitives may be added
- Existing semantics remain stable
- Breaking changes require version increments

Future scripting or programmable behavior, if introduced, will:
- Exist as a separate system
- Consume Lucius outputs
- Preserve DSL determinism

---

## Summary

`lyara!` is a **restricted, intentional language** designed for:
- Safety over expressiveness
- Clarity over cleverness
- Trust over magic

It encodes **constraints**, not opinions.

### Example

```rust
lyara! {
    meta {
        name        = "suspicious_pdf_active_content"
        author      = "org-security"
        source      = "internal-research"
        version     = "1.2.0"

        // scoring semantics
        base_weight = 0.15
        severity    = high

        // downstream handling
        forensics   = true
    }

    actions {
        quarantine
        extract_strings
        request_deep_scrub
        notify_soc
    }

    rules {
        pdf_magic = bytes {
            value  = { 25 50 44 46 }   // %PDF
            offset = 0,
            applies_to = binary
        }

        embedded_javascript = ascii {
            contains = "JavaScript"
            nocase   = true
            applies_to = webpage
        }

        launch_action = ascii {
            any_of = [
                "/Launch",
                "/OpenAction",
                "/AA"
            ]
        }

        entropy_spike = entropy {
            range = 7.6..8.0
        }

        shellcode_stub = bytes {
            value = { 90 90 ?? E8 ?? ?? ?? ?? }
        }

        //ideally can pull from file or list
        known_bad_hash = hash {
            sha256 = [
                "b1946ac92492d2347c6235b4d2611184",
                "e3b0c44298fc1c149afbf4c8996fb924"
            ]
        }
    }

    conditions {
        // Structural validation
        when not pdf_magic {
            score += 0.9
            tag   += "spoofed-extension"
        }

        // Active content detection
        when all(pdf_magic, embedded_javascript) {
            score += 0.4
            tag   += "active-content"
        }

        when launch_action {
            score += 0.3
            tag   += "auto-exec"
        }

        // Heuristic suspicion
        when entropy_spike count 2..5 {
            score += 0.25
            tag   += "compressed-or-obfuscated"
        }

        // Strong indicators
        when shellcode_stub {
            score += 0.6
            tag   += "shellcode-pattern"
            signal     += YaraShellCode
            risk_hint  += ActiveContent
        }

        when known_bad_hash {
            score += 1.0
            tag   += "known-malware"
        }

        // Decision thresholds
        when score >= 0.5 {
            enact extract_strings
        }

        when score >= 0.7 {
            enact quarantine
            enact notify_soc
        }

        when score >= 0.85 {
            enact request_deep_scrub
        }
    }
}
```