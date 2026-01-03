### lfin Configuration Macro - Purpose and Scope

The `lfin` configuration macro defines how Lucius *interprets* accumulated evidence. It is the final reasoning layer in the Lucius pipeline, responsible for turning signals, statistical outputs, and contextual hints into a coherent, explainable outcome.

At a high level, `lfin` answers the question:

> “Given everything Lucius observed, how should this artifact be understood?”

#### What lfin does

- Assigns meaning to **signals** emitted by upstream components  
  Signals are factual observations (e.g. `PdfHasJavascript`, `KnownBadHash`).  
  `lfin` determines how much each signal matters by assigning:
  - Weight
  - Confidence impact
  - Whether the signal should be surfaced to Ben or other systems

- Interprets **risk hints**  
  Risk hints bias interpretation without asserting maliciousness.  
  They shape severity, confidence caps, and contextual framing rather than making decisions on their own.

- Correlates weak or strong evidence  
  `lfin` allows combinations of signals to reinforce, dampen, or override each other so that meaning emerges from context rather than single indicators.

- Governs confidence  
  `lfin` enforces confidence rules to prevent overconfidence when data is missing, partial, or contradictory. Confidence is treated as a first-class output.

- Defines decision thresholds  
  Decisions such as *observe*, *investigate*, or *contain* are derived from accumulated score and confidence according to operator-defined ranges.

- Declares bounded operation permissions  
  `lfin` specifies which operations are allowed or forbidden for each decision state. These are declarative permissions, not executions.

- Normalizes output  
  `lfin` controls what information is persisted in the Lucius silo and what is returned upstream (e.g. to Ben), including redaction and verbosity.

- Defines failure posture  
  Explicit behavior is declared for degraded conditions such as missing data, feed failures, or internal errors.

#### What lfin is not

- It is not a detector
- It does not perform analysis
- It does not execute operations
- It does not assert ground truth maliciousness

`lfin` is an interpretation engine, not an enforcement engine.

#### Authority model

`lfin` is authoritative over interpretation but bounded by declared intent.  
It cannot invent signals, exceed configured operations, or hide uncertainty.

All conclusions produced by `lfin` are explainable, reproducible, and versioned.

#### Design goal

The goal of `lfin` is to replace opaque yes/no verdicts with structured, reasoned outcomes that operators can trust, audit, and integrate into larger decision systems.





### Example
```rust
lfin! {
    meta {
        name        = "default_enterprise_triage"
        version     = "0.3.0"
        author      = "org-security"
        intent      = Investigative
        description = "Balanced interpretation profile for mixed enterprise workloads"
    }

    // Signal semantics
    //
    // Signals are factual observations emitted by upstream components.
    // lfin assigns meaning, weight, and confidence impact.
    signals {
        PdfHasJavascript {
            weight        = 0.35
            confidence    = 0.9
            export_to_ben = true
        }

        ShellcodePattern {
            weight        = 0.75
            confidence    = 0.95
            export_to_ben = true
        }

        KnownBadHash {
            weight        = 1.0
            confidence    = 1.0
            export_to_ben = true
        }

        SuspiciousEntropy {
            weight        = 0.25
            confidence    = 0.6
            export_to_ben = false
        }
    }

    // Risk hints
    //
    // Risk hints bias interpretation but never assert maliciousness.
    // They shape thresholds, not decisions.
    risk_hints {
        ActiveContent {
            severity_bias = +0.15
        }

        ObfuscationLikely {
            severity_bias = +0.2
        }

        UnknownFormat {
            severity_bias = +0.3
            confidence_cap = 0.7
        }
    }

    // Composite interpretation rules
    //
    // Allows weak signals to reinforce or dampen each other.
    correlations {
        when all(PdfHasJavascript, SuspiciousEntropy) {
            amplify_weight += 0.2
            note "Active content combined with entropy anomaly"
        }

        when any(ShellcodePattern, KnownBadHash) {
            override_confidence = 1.0
        }

        when SuspiciousEntropy and not ActiveContent {
            dampen_weight -= 0.1
        }
    }

    // Confidence governance
    //
    // Prevents overconfidence when data is partial or conflicting.
    confidence_rules {
        minimum_required = 0.6

        when missing(StructuralAnalysis) {
            cap_confidence = 0.7
        }

        when conflicting_signals {
            downgrade_severity = one_level
        }
    }

    // Decision thresholds
    //
    // Thresholds operate on accumulated score + confidence.
    thresholds {
        observe {
            score_range      = 0.0..0.4
            confidence_min   = 0.0
        }

        investigate {
            score_range      = 0.4..0.7
            confidence_min   = 0.5
        }

        contain {
            score_range      = 0.7..1.0
            confidence_min   = 0.75
        }
    }

    // Bounded Operations
    //
    // operations are explicit, finite, and non-escalatory.
    operations {
        when decision == investigate {
            allow extract_strings
            allow enrich_metadata
            deny  quarantine
        }

        when decision == contain {
            allow quarantine
            allow notify_soc
            allow request_luxamen
        }
    }

    // Output normalization
    //
    // Shapes what is persisted and what is returned upstream.
    output {
        lucius_silo {
            verbosity = detailed
            include   = [signals, risk_hints, confidence, decision]
        }

        ben {
            emit_signals = true
            emit_decision = true
            redact_tags  = false
        }
    }

    // Failure posture
    //
    // Explicit behavior under uncertainty or degradation.
    failure {
        on_missing_data   = degrade_confidence
        on_feed_failure   = annotate_only
        on_internal_error = halt_and_record
    }
}
```