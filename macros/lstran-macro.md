
# `lstran!` - Lucius Structural Analysis DSL

This document explains the **`lstran!` macro** using a concrete example.  
`lstran!` defines **structural analysis intent** for Lucius: fast, bounded, deterministic inspection that establishes *what an artifact is*, *how confident we are*, and *what structural signals it exhibits* without asserting final maliciousness.

Structural analysis exists to **reduce ambiguity early**, **route work intelligently**, and **preserve throughput** in adversarial environments.

---

## What `lstran!` Is

`lstran!` is a **declarative, compile-time validated declarative proc macro utilizing a DSL** and is used to describe:

- Artifact classification (observed type vs claimed type)
- Structural probes (PDF, PE, archive, text, etc.)
- Bounded parsing behavior
- Signals and tags derived from structure
- Routing *hints* for later stages (YARA profiles, static analysis candidacy)

It is **not** a scoring component, **not** a malware verdict system, and **not** a general-purpose parser framework.

Its output is a **Structural Report** consumed by:
- The **Notary** (for accumulation and audit)
- The **Assessor** (for static analysis eligibility)
- The **YARA component** (for rule selection)
- Finalization and statistics layers

---

## Design Intent

Structural analysis answers questions like:

- *What does this artifact actually appear to be?*
- *Does its structure contradict its claims?*
- *Are there features that warrant deeper inspection?*
- *Which downstream analyses make sense?*

It does **not** attempt to:
- Fully decompile
- Emulate execution
- Replace static or dynamic analysis
- Assert maliciousness as a final truth

---

## High-Level Shape

An `lstran!` macro is composed of these sections:

| Block | Purpose |
| :--- | :--- |
| **meta** | Identity and audit metadata |
| **bounds** | Hard execution limits (determinism contract) |
| **magic** | Low-cost signature checks |
| **parse** | Bounded structural probes |
| **emit** | Normalized outputs |
| **conditions** | Logic that derives signals and routing hints |


Each block has **explicit responsibilities and boundaries**.

---

## `meta` - Identity and Audit

```rust
meta {
    name       = "lucius_structural_default"
    author     = "org-security"
    source     = "internal-policy"
    version    = "0.3.0"
    applies_to = any
    profile    = "hot-path"
}
```
### Purpose
- Provides stable identity for versioning and replay
- Aids auditing and reasoning
- Does not affect execution logic directly

profile is informational intent (e.g., hot-path vs deep-path), not enforcement.

---

## bounds - Determinism and Throughput Guarantees

```rust
bounds {
    max_read_bytes        = 8.mib
    max_scan_bytes        = 256.kib
    max_container_depth   = 4
    max_container_members = 64
    max_member_read_bytes = 256.kib
    max_text_parse_bytes  = 512.kib
    fail_mode             = fail_soft
}
```
### Purpose

Bounds are **non-negotiable** invariants.

They ensure:
- Predictable cost
- Resistance to adversarial payloads
- No accidental DoS from parsing logic

If bounds are exceeded:
- Lucius emits explicit signals
- Analysis continues conservatively
- No silent failure, no false confidence

Structural analysis never pretends certainty when budgets are exhausted.
---

## magic  - Cheap Structural Hints

```rust
magic {
    pdf_magic = bytes { offset = 0 value = { 25 50 44 46 } }
    mz_magic  = bytes { offset = 0 value = { 4D 5A } }
    zip_magic = bytes { offset = 0 value = { 50 4B 03 04 } }
    html_hint = ascii { any_of = ["<html", "<script"] nocase = true }
}
```
### Purpose
- Extremely low-cost checks
- Establish early identity
- Avoid unnecessary parsing

Magic checks **do not** imply trust -only observation.

---

## parse - Bounded Structural Probes
```rust
parse {
    pdf_probe = pdf {
        detect_features = ["has_javascript", "has_openaction"]
        max_objects = 10_000
    }

    pe_probe = pe {
        detect_features = ["has_tls_callbacks", "has_overlay"]
        max_sections = 96
    }

    archive_probe = archive {
        formats = [zip, tar, gzip]
        classify_by_members = true
    }
}
```

### Purpose

**Parsers are**:
- Purpose-built
- Non-extensible by user code
- Bounded by bounds

**They answer structural questions only**:
- Features present
- Partial vs complete parse
- Container composition

No execution, no emulation, no scripting.

---

## emit  - Normalized Outputs

```rust
emit {
    field yara_class     : YaraClass
    field observed_type  : ObservedType
    field type_confidence: Confidence
    field parse          : ParseSummary
    tags
    signals
    risk_hints
}
```
### Purpose

**Defines the contract** between structural analysis and the rest of Lucius.
- Tags: human-readable labels
- Signals: typed, machine-consumable observations
- Risk hints: enums, not scores

Scoring is **deferred** to later stages.

---
## conditions  - Derivation Logic

Conditions look similar to lyara!, but their role is different.

### Example: Type Establishment

```rust
when pdf_magic {
    observed_type   = pdf
    yara_class      = document
    type_confidence = high
    tag += "type:pdf"
}
```
**This:**
- Establishes observed type
- Assigns YARA class for downstream selection
- Records confidence explicitly

---
## Example: Structural Mismatch

```rust
when ctx.claimed_ext != observed_type {
    signal += ClaimedTypeMismatch
    risk_hint += SpoofedExtension
}
```
Mismatch is:
- A signal
- Not a verdict

---
## Example: Feature-Based Signals
```rust
when parse.pdf.has_javascript {
    signal += PdfHasJavascript
    risk_hint += ActiveContent
}
```
**Signals accumulate context**; they do not decide outcomes.
---
## Example: Routing Hints

```rust
when yara_class == document {
    signal += YaraProfileHint { profiles: ["pdf_common"] }
}
```
**Routing hints:**
- Inform downstream components
- Are non-binding
- Preserve operator intent

---
## What lstran! Explicitly Does Not Do
- No scoring
- No malware verdicts
- No dynamic execution
- No user-defined parsers
- No unbounded recursion
- No hidden heuristics

Every behavior is visible, bounded, and versioned.

---
## Relationship to Other Components
- YARA (lyara!)
- Uses yara_class and hints to select rule sets
- Assessor
- Uses structural signals to decide static analysis eligibility
- Notary
- Accumulates signals, tags, and parse outcomes
- Finalize
- Converts accumulated evidence into human-readable outputs

Structural analysis exists to make everything else cheaper, clearer, and more honest.
---
Summary

```lstran!``` is the first truth-finding stage in Lucius.

**It:**
- Reduces ambiguity
- Preserves determinism
- Exposes uncertainty
- Enables intelligent routing
- Avoids premature conclusions

Structural analysis is **not** about being right.
it is about being explicit, bounded, and useful.

---
### Example

```rust
// lstran! - Lucius Structural Analysis DSL (example)
// Goal: fast, deterministic classification + structural descriptors (signals), not final scoring.
// Output: a StructuralReport (signals/tags/observations) that Notary + Assessor consume.
//
// Notes:
// - No arbitrary parsing code.
// - No external file includes.
// - Bounded execution via explicit limits.
// - Emits *signals*; optional risk hints are enums, not floats.

lstran! {
    meta {
        name        = "lucius_structural_default"
        author      = "org-security"
        source      = "internal-policy"
        version     = "0.3.0"

        // High-level applicability. Used for organization + auditing, not gating.
        // (Gating is done via conditions + routing hints.)
        applies_to  = any

        // Determinism + throughput intent for this profile.
        profile     = "hot-path"
    }

    // Hard bounds that every detector and parser must respect.
    // This is your "no surprise costs" contract.
    bounds {
        // Maximum bytes read from the artifact total across all checks.
        max_read_bytes         = 8.mib

        // Maximum bytes used for signature scanning.
        max_scan_bytes         = 256.kib

        // Maximum recursion for nested containers (zip-in-zip-in-zip).
        max_container_depth    = 4

        // Maximum total extracted/inspected members across all containers.
        max_container_members  = 64

        // Per-member inspection cap (e.g., first N bytes of each file in a zip).
        max_member_read_bytes  = 256.kib

        // Maximum parsing effort for “structured text” (JSON/XML).
        max_text_parse_bytes   = 512.kib

        // If exceeded, we *fail-soft* in structural analysis:
        // emit bounded signals and route for deeper inspection rather than pretending certainty.
        fail_mode              = fail_soft
    }

    // Signature checks: quick, deterministic checks before any heavier parsing.
    // "magic" checks can set observed type/class quickly.
    magic {
        pdf_magic = bytes {
            offset = 0
            value  = { 25 50 44 46 } // %PDF
        }

        mz_magic = bytes {
            offset = 0
            value  = { 4D 5A } // MZ (PE / DOS)
        }

        elf_magic = bytes {
            offset = 0
            value  = { 7F 45 4C 46 } // \x7FELF
        }

        zip_magic = bytes {
            offset = 0
            value  = { 50 4B 03 04 } // PK..
        }

        gzip_magic = bytes {
            offset = 0
            value  = { 1F 8B } // gzip
        }

        png_magic = bytes {
            offset = 0
            value  = { 89 50 4E 47 0D 0A 1A 0A }
        }

        // Useful for “web-ish” payloads even if extension lies.
        html_hint = ascii {
            // bounded scan; respects bounds.max_scan_bytes
            any_of = [ "<!DOCTYPE html", "<html", "<script", "<meta" ]
            nocase = true
        }

        // Basic UTF-8 plausibility (not full validation).
        utf8_plausible = text {
            encoding = utf8
            // heuristics
            max_invalid_ratio = 0.02
            max_control_ratio = 0.08
        }
    }

    // Built-in parsers. These are not general purpose, they’re “structural probes.”
    // Each parser returns a ParseState + descriptors.
    parse {
        pdf_probe = pdf {
            // Probe only; no full render, no JS execution, no object decompression bombs beyond bounds.
            detect_features = [
                "has_javascript",
                "has_embedded_files",
                "has_openaction",
                "has_launch",
                "has_xfa",
                "has_encryption"
            ]
            max_objects = 10_000
        }

        pe_probe = pe {
            detect_features = [
                "is_dll",
                "is_driver",
                "has_overlay",
                "has_debug_dir",
                "has_tls_callbacks",
                "has_imports",
                "has_exports",
                "has_rsrc"
            ]
            max_sections = 96
        }

        archive_probe = archive {
            // Container introspection only.
            formats = [ zip, tar, gzip ]
            // Normalize nested archive identity: zip->docx, zip->jar, etc.
            classify_by_members = true
        }

        json_probe = json {
            // Just: is it valid-ish JSON, and if so what shape?
            detect_features = [ "is_ndjson", "top_level_kind", "key_count_estimate" ]
        }
    }

    // Normalized output fields produced by structural analysis.
    // Think “facts we can carry forward,” not “final judgment.”
    emit {
        // Canonical class for later pipeline gating (YARA profile selection, static analysis eligibility).
        // This is a hint; downstream still uses intent + evidence.
        field yara_class : YaraClass

        // Observed structural type.
        field observed_type : ObservedType

        // How confident is the observed type?
        field type_confidence : Confidence // { low, medium, high }

        // Structural parse state for each probe.
        field parse : ParseSummary

        // Tags: strings for humans and search; stable-ish but versioned by DSL.
        tags

        // Signals: typed observations; machine-friendly.
        signals

        // Risk hints: enums, not floats. Notary/Finalize can map them to weights.
        risk_hints
    }

    // Conditions: like lyara, but focused on classification + signals + routing hints.
    conditions {
        // --- Type identification and mismatch signaling ---

        when pdf_magic {
            observed_type    = pdf
            yara_class       = document
            type_confidence  = high
            tag += "type:pdf"
        }

        when mz_magic {
            observed_type    = pe
            yara_class       = binary
            type_confidence  = high
            tag += "type:pe"
        }

        when elf_magic {
            observed_type    = elf
            yara_class       = binary
            type_confidence  = high
            tag += "type:elf"
        }

        when zip_magic {
            observed_type    = archive_zip
            yara_class       = archive
            type_confidence  = high
            tag += "type:zip"
        }

        when gzip_magic and not zip_magic {
            observed_type    = archive_gzip
            yara_class       = archive
            type_confidence  = medium
            tag += "type:gzip"
        }

        // If it smells like HTML or script content, classify as web-ish.
        when html_hint and utf8_plausible and not any(pdf_magic, mz_magic, elf_magic, zip_magic) {
            observed_type    = web_text
            yara_class       = webpage
            type_confidence  = medium
            tag += "type:web"
            signal += WebTextDetected { indicators: ["html/script/meta"] }
        }

        // If nothing matches, stay honest.
        when not any(pdf_magic, mz_magic, elf_magic, zip_magic, gzip_magic, png_magic, html_hint) {
            observed_type    = unknown
            yara_class       = unknown
            type_confidence  = low
            tag += "type:unknown"
            signal += UnknownType
        }

        // Extension mismatch is a signal, not a verdict.
        // (Assumes Normalize stage provided a claimed_ext into the context.)
        when ctx.claimed_ext is_some and ctx.claimed_ext != observed_type {
            tag    += "mismatch:claimed-vs-observed"
            signal += ClaimedTypeMismatch {
                claimed: ctx.claimed_ext,
                observed: observed_type
            }
            risk_hint += SpoofedExtension
        }

        // --- Probe execution rules (bounded) ---

        // Probe PDF only if it’s actually PDF.
        when observed_type == pdf {
            parse.pdf = run pdf_probe
        }

        // Probe PE only if it’s PE.
        when observed_type == pe {
            parse.pe = run pe_probe
        }

        // Probe archives only if archive-like.
        when yara_class == archive {
            parse.archive = run archive_probe
        }

        // Probe JSON only if it’s plausibly text and within budget.
        when utf8_plausible and ctx.size_bytes <= 2.mib and not any(pdf_magic, mz_magic, elf_magic) {
            parse.json = run json_probe
        }

        // Signals from probe outputs 

        // PDF feature signals
        when parse.pdf.state == ok and parse.pdf.has_javascript {
            tag    += "pdf:js"
            signal += PdfHasJavascript
            risk_hint += ActiveContent
        }

        when parse.pdf.state == ok and parse.pdf.has_openaction {
            tag    += "pdf:openaction"
            signal += PdfHasOpenAction
            risk_hint += AutoExecVector
        }

        when parse.pdf.state == ok and parse.pdf.has_embedded_files {
            tag    += "pdf:embedded"
            signal += PdfHasEmbeddedFiles { count: parse.pdf.embedded_count }
            risk_hint += NestedPayload
        }

        when parse.pdf.state == partial {
            tag    += "pdf:partial-parse"
            signal += PartialParse { kind: pdf }
            risk_hint += StructuralAnomaly
        }

        // PE feature signals
        when parse.pe.state == ok and parse.pe.has_tls_callbacks {
            tag    += "pe:tls-callbacks"
            signal += PeHasTlsCallbacks
            risk_hint += SuspiciousExecutionFlow
        }

        when parse.pe.state == ok and parse.pe.has_overlay {
            tag    += "pe:overlay"
            signal += PeHasOverlay { size: parse.pe.overlay_size }
            risk_hint += PackedOrModified
        }

        when parse.pe.state == partial {
            tag    += "pe:partial-parse"
            signal += PartialParse { kind: pe }
            risk_hint += StructuralAnomaly
        }

        // Archive signals: nested depth, suspicious compositions, type normalization
        when parse.archive.state == ok and parse.archive.depth > 1 {
            tag    += "archive:nested"
            signal += NestedContainer { depth: parse.archive.depth }
            risk_hint += NestedPayload
        }

        when parse.archive.state == ok and parse.archive.classification == docx {
            // A docx is a zip with specific members.
            observed_type   = docx
            yara_class      = document
            type_confidence = high
            tag += "type:docx"
        }

        when parse.archive.state == ok and parse.archive.classification == jar {
            observed_type   = jar
            yara_class      = binary
            type_confidence = high
            tag += "type:jar"
        }

        when parse.archive.state == partial {
            tag    += "archive:partial-parse"
            signal += PartialParse { kind: archive }
            risk_hint += StructuralAnomaly
        }

        // Bound exhaustion: fail-soft behavior emits “budget exceeded” signal
        when ctx.bounds_exceeded {
            tag    += "bounds:exceeded"
            signal += BoundsExceeded {
                max_read_bytes: bounds.max_read_bytes,
                max_depth: bounds.max_container_depth
            }
            // This is a strong reason to consider deeper inspection, but not an automatic accusation.
            risk_hint += AnalysisIncomplete
        }

        // Routing hints (NOT decisions) 

        // Suggest which YARA profile families are relevant.
        when yara_class == document {
            signal += YaraProfileHint { profiles: ["doc_common", "pdf_common"] }
        }

        when yara_class == webpage {
            signal += YaraProfileHint { profiles: ["web_common", "phish_common"] }
        }

        when yara_class == binary and observed_type == pe {
            signal += YaraProfileHint { profiles: ["pe_common", "packer_common"] }
        }

        when yara_class == archive {
            signal += YaraProfileHint { profiles: ["archive_common", "nested_common"] }
        }

        // Suggest static analysis candidacy (Assessor decides based on intent).
        when any(
            risk_hint contains ActiveContent,
            risk_hint contains AutoExecVector,
            risk_hint contains SuspiciousExecutionFlow,
            risk_hint contains PackedOrModified,
            risk_hint contains AnalysisIncomplete
        ) {
            signal += StaticCandidateHint { reason: "structural-risk-hints" }
        }
    }
}
```