# Implementation Plan: Rules Engine & Parser




## Overview




The implementation strategy follows a domain-driven approach, prioritizing the definition of the Rule and Violation models before implementing the execution logic. We start by building a robust Domain Layer to house the Rule Registry and Severity definitions, ensuring a unified interface for style, logic, and security rules. This provides the foundational 'contract' that both the parser and the runner must satisfy.

The second phase implements the AST Parser using LibCST to ensure lossless source-to-AST conversion, supporting the Parse Fidelity Invariant. Finally, we implement the Parallel Rule Runner using a process-based execution model to meet performance requirements for CI/CD environments. This sequence ensures that we have a stable schema and parsing capability before introducing the complexity of multi-processed execution.




## Tasks




- [ ] F0a-T1. Domain Layer: Rules and Registry

- [ ] F0a-T1.1 Define Core Rule Models and Registry

- Create `src/domain/rules/models.py` defining `Severity` (Enum: INFO, MEDIUM, CRITICAL) and `Violation` (Dataclass).

- Define the `Rule` abstract base class with an `evaluate(tree: CST) -> List[Violation]` interface.

- Implement the `RuleRegistry` in `src/domain/rules/registry.py` with methods for `register()`, `get_rules_by_category()`, and `all_rules()`.

- Verification: Unit test the registry to ensure rules for security, logic, and style can be retrieved independently.

- _Requirements: 3_


- [ ] F0a-T1.2 Implement Multi-Domain Rule Templates

- Create `src/domain/rules/definitions/` package.

- Implement a placeholder `CriticalSecurityRule` and `StandardStyleRule` to facilitate testing.

- Ensure `RuleMetadata` includes unique IDs and impact scores.

- _Requirements: 3, 4_



- [ ] F0a-T2. Adapter Layer: AST Parsing

- [ ] F0a-T2.1 Implement Lossless AST Parser Adapter

- Create `src/adapters/parser/libcst_adapter.py`.

- Implement `LibCSTParser` class with a `parse(source: str) -> libcst.Module` method.

- Implement a `to_source(tree: libcst.Module) -> str` method for round-trip verification.

- Verification: Compare original file content with serialized AST output.

- _Requirements: 1_


- [ ] F0a-T2.2 Property Test: Parse Fidelity

- Write property-based tests using Hypothesis.

- Generate random Python strings and assert that `parse(to_source(tree))` is an identity function.

- _Requirements: 1_



- [ ] F0a-T3. Use Case Layer: Parallel Execution Engine

- [ ] F0a-T3.1 Implement Parallel Rule Runner

- Create `src/usecases/engine/runner.py`.

- Implement `ParallelRunner` using `concurrent.futures.ProcessPoolExecutor`.

- Implement `run_on_file(path: Path)` which coordinates parsing and rule application.

- Implement logic to aggregate results and filter for CRITICAL violations to support CI blocking.

- _Requirements: 2, 4_


- [ ] F0a-T3.2 Property Test: Parallel Efficiency

- Create a performance benchmark script in `tests/benchmarks`.

- Measure T_parse, T_rules_max, and total T(N) for 100+ files.

- Assert that T(N) < 1.5 * (T_parse + T_rules_max).

- _Requirements: 2_


- [ ] F0a-T3.3 Property Test: Security Violation Invariant

- Implement a test case with a known 'CRITICAL' vulnerability rule.

- Verify that the `ParallelRunner` returns the violation and triggers an exit code 1 (blocking).

- _Requirements: 4_



- [ ] F0a-T4. Final Integration & Checkpoint

- [ ] F0a-T4.1 Checkpoint: System Integration & Correctness

- Run the full suite of unit and property tests.

- Verify that the Rule Registry correctly categorizes rules into Style, Logic, and Security buckets.

- Ensure serialized ASTs are bit-for-bit identical to source files.

- _Requirements: 1, 2, 3, 4_




## Notes




- Tasks marked with (*) in the title are optimizations that can be deferred if critical path velocity is prioritized.

- Traceability is maintained by linking every sub-task to specific Requirements (R1-4) or Properties (P1-3).

- Ordering follows a Domain-First approach: we define what a 'Rule' and 'Violation' are before building the machinery to execute them.

- LibCST is selected for the adapter to satisfy the Parse Fidelity Invariant (P3) due to its full-lossless tree representation.

- The use of ProcessPoolExecutor is mandated to bypass the Global Interpreter Lock (GIL) for true parallel execution (R2).

