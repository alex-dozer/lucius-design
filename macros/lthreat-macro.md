
# lthreat Macro - Summary

The `lthreat!` macro defines how Lucius interacts with external threat intelligence sources in a **bounded, deterministic, and auditable** way.

Rather than performing detection or decision-making, `lthreat!` exists to **corroborate artifacts** against known threat feeds and translate those results into **explicit statistical contributions** that downstream components can reason about.

---

## Purpose

The macro allows operators to declaratively specify:

- Which threat feeds are consulted  
- What hash types are supported  
- How much authority each feed is granted  
- How failures are handled  
- How confirmed matches influence scoring and metadata  

This ensures that threat intelligence is treated as **contextual input**, not unquestioned truth.

---

## Design Principles

- **Declarative, not procedural**  
  The macro expresses *policy and trust*, not execution logic or branching behavior.

- **Deterministic**  
  Given the same artifact, feed configuration, and response, the outcome is repeatable and explainable.

- **Failure-aware**  
  Feed outages, timeouts, and partial responses are handled explicitly and never silently ignored.

- **Non-authoritative**  
  Threat feeds contribute weight and context, but never directly determine maliciousness or actions.

---

## What the Macro Defines

Each feed declaration specifies:

- **Feed identity** and supported hash algorithms  
- **Authority semantics**, including baseline confidence and scope  
- **Failure behavior**, such as whether to continue, defer, or degrade confidence  
- **Match behavior**, including score adjustments, tags, and optional structured metadata capture  

All contributions are accumulated and recorded for later reasoning by downstream components.

---

## What the Macro Does *Not* Do

The `lthreat!` macro explicitly does **not**:

- Perform detection or classification  
- Execute actions or trigger enforcement  
- Perform fuzzy matching or probabilistic inference  
- Hide uncertainty or normalize feed quality  
- Replace internal analysis or operator intent  

---

## Role in the System

`lthreat!` integrates cleanly into the Lucius pipeline:

- Hashes are computed upstream  
- Feeds are queried according to declared policy  
- Results are logged and weighted  
- Final interpretation is deferred to later stages  

This separation ensures that external intelligence enhances understanding without eroding control or transparency.



### Example

```rust
lthreat! {
    feed "internal_malware_feed" {
        provider = "org-threat-intel"
        hashes   = [ sha256, sha1 ]

        authority {
            confidence = 0.9
            scope      = internal
        }

        on_failure {
            behavior = record_and_continue
            penalty  = 0.0
        }

        on_match {
            score += 0.6
            tag   += "known-malware"
            tag   += "threat-feed-match"

            capture {
                family
                campaign
                first_seen
            }
        }
    }

    feed "public_reputation_feed" {
        provider = "reputation-api"
        hashes   = [ sha256 ]

        authority {
            confidence = 0.5
            scope      = external
        }

        on_failure {
            behavior = degrade_confidence
            penalty  = 0.1
        }

        on_match {
            score += 0.3
            tag   += "public-malware-indicator"
        }
    }
}
```