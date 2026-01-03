## Non-Goals

Lucius is deliberately scoped. The following are explicit non-goals of the system.

- **Catching every malicious artifact**
  
  Lucius does not claim exhaustive detection. Adversarial systems evolve, heuristics decay, and perfect coverage is neither realistic nor honest.

- **Hand-holding operators**
  
  Lucius assumes a technically competent operator. It surfaces evidence, reasoning, and outcomes, but does not attempt to abstract away the need for judgment.

- **Assuming the system knows better than the organization**
  
  Lucius does not override declared intent, organizational risk tolerance, or operational constraints in favor of vendor-defined or model-defined “best practices.”

- **Acting as a detonation or VM orchestration engine**
  
  Lucius is not a sandbox or detonation platform. Execution, emulation, and VM lifecycle management are explicitly out of scope and delegated to Luxamen or equivalent systems.

- **Supporting every YARA engine or dialect**
  
  Lucius does not aim to be universally compatible with all YARA implementations. Only constrained, explicitly supported semantics are allowed where they preserve correctness and throughput.

- **Offering universal extensibility across all decompilers and analyzers**
  
  Lucius does not attempt to integrate every possible static analysis or reverse engineering tool. Integration is deliberate and bounded.

- **Trafficking in probabilistic vibes disguised as certainty**
  
  Lucius does not emit confidence without attributable reasoning. When it asserts maliciousness, that assertion is grounded in explicit rules, structural indicators, declared intent, and observable evidence; not intuition, opacity, or model affect.

- **Replacing operator judgment**
  
  Lucius exists to inform and constrain decisions, not to absolve operators of responsibility or critical thinking.