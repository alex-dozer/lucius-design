### lstatic macro

Lucius deliberately separates **knowledge**, **mechanism**, and **intent**.

Static analysis signatures (API names, strings, opcode patterns, hashes, etc.) are **not embedded in configuration macros**. These datasets are unbounded, frequently updated, organization-specific, and best treated as **operational knowledge**, not policy. Encoding them in DSLs would introduce duplication, brittleness, and long-term maintenance debt.

Instead:

- **Static analysis engines** perform limited, deterministic inspection (Tier 1) and emit **typed signals** (e.g. `imports::memory_injection`, `binary::packed`, `script::dynamic_eval`).
- **Knowledge stores / adapters** own signature data, feeds, and mappings, and translate raw observations into standardized signals.
- **Configuration macros** (e.g. `lstatic!`) express **interpretation, composition, thresholds, and routing**, not raw signatures.

The DSL exists to answer:
- How signals combine
- How they affect scoring
- When escalation occurs
- When deeper analysis is justified

This design:
- Preserves determinism and throughput
- Avoids encoding volatile data as policy
- Keeps operator intent authoritative
- Allows independent evolution of datasets and analysis engines
- Reduces complexity for a solo maintainer


### Example
```rust
lstatic! {
    meta {
        name    = "binary_semantic_interpretation"
        version = "0.1.0"
        scope   = binary
    }

    context {
        platform = any
        risk     = adversarial
    }

    conditions {
        when signal.imports.memory_injection {
            score += 0.35
            tag   += "memory-injection-capability"
        }

        when signal.binary.packed {
            score += 0.25
            tag   += "obfuscated"
        }

        when all(
            signal.imports.memory_injection,
            signal.binary.packed
        ) {
            score += 0.30
            tag   += "likely-loader"
        }

        when signal.script.dynamic_eval {
            score += 0.20
            tag   += "runtime-code-generation"
        }

        when score >= 0.6 {
            enact route_to_assessor
        }

        when score >= 0.8 {
            enact request_deep_scrub
        }
    }
}
```