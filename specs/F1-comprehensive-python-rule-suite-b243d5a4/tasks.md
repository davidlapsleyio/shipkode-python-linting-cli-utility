# Implementation Plan: Comprehensive Python Rule Suite




## Overview




The implementation strategy follows a domain-driven approach, prioritizing the core data models and rule interfaces before building the high-performance execution engine and UI. We will first define the 'Issue' and 'Report' domain objects to ensure a consistent contract between the engine and the adapters. This is followed by implementing the specific PEP 8 and Security rule logic, then the parallelized engine, and finally the rich TUI reporter.

The rationale for this ordering is to establish 'Aggregation Integrity' early. By defining how issues are collected and summed before implementing parallel workers, we reduce the risk of bug propagation in the multi-processing logic. Incremental processing is implemented last as an optimization layer over a stable, functional linting engine.




## Tasks




- [ ] F1-T1. Domain Model Definition

- [ ] F1-T1.1 Define Domain Issue and Report Models

- Create `src/domain/models.py`

- Define `Issue` dataclass with fields: rule_id, line, column, message, severity (Enum: STYLE, SECURITY, LOGIC).

- Define `Report` dataclass with dictionary for aggregation and summary statistics.

- _Requirements: 1, 2_


- [ ] F1-T1.2 Define Rule and Reporter Interfaces

- Create `src/domain/interfaces.py`

- Define `BaseRule` abstract class with a `check(path: Path) -> List[Issue]` method.

- Define `IReporter` interface for UI output.

- _Requirements: 1, 2_



- [ ] F1-T2. Rule Suite Implementation

- [ ] F1-T2.1 Implement PEP 8 Governance Rules

- Implement `PEP8StyleRule` in `src/adapters/rules/suite.py` using the `pycodestyle` or `flake8` internal APIs.

- Ensure violation SEVERITY_STYLE is correctly mapped.

- Verification: Create a sample file with E302 violations and run unit tests.

- _Requirements: 1_


- [ ] F1-T2.2 Implement Security and Logic Rules

- Implement `SecurityVulnerabilityRule` using `bandit` or regex-based pattern matching for insecure code (e.g., eval(), subprocess.shell=True).

- Implement `LogicalErrorRule` for common anti-patterns like broad except blocks.

- _Requirements: 2_



- [ ] F1-T3. Engine and Use Case Development

- [ ] F1-T3.1 Implement Parallel Linting Engine

- Create `src/usecases/engine.py`.

- Implement `LintingEngine` that accepts a list of rules and file paths.

- Use `concurrent.futures.ProcessPoolExecutor` for parallel execution.

- Implement result aggregation logic.

- _Requirements: 4_


- [ ] F1-T3.2 Add Incremental Linting Support (*)

- Implement file-hashing logic in `src/usecases/engine.py` to track `mtime` and content hashes.

- Store state in a local `.linter_cache` file.

- Logic: Skip `check()` call if file hash matches the cache entry.

- _Requirements: 4_



- [ ] F1-T4. Checkpoint: Core Logic Verification

- [ ] F1-T4.1 Execute Correctness Property Tests

- Run property tests for 'Aggregation Integrity' by comparing single-threaded vs multi-threaded results.

- Run property tests for 'Incremental Efficiency' by checking file access counts on repeated runs.



- [ ] F1-T5. TUI and Reporting UI

- [ ] F1-T5.1 Implement Rich TUI Reporter

- Create `src/adapters/ui/tui.py`.

- Integrate `rich` library for table rendering and progress bars.

- Implement color-coding based on severity (Red for Security, Yellow for Style).

- _Requirements: 3_


- [ ] F1-T5.2 Final CLI Integration

- Finalize `src/main.py` to wire up the Engine, Suite, and TUI.

- Ensure the summary table displays total counts for E-codes and security levels.

- _Requirements: 3_




## Notes




- Tasks marked with (*) are optional optimizations for CI/CD environments. Traceability is maintained via requirement_refs to ensure every user story is addressed. The domain-first ordering ensures that the data structures for 'Issues' and 'Reports' are stable before the parallel engine or UI logic is built. Backward compatibility for existing rule definitions is guaranteed by the interface-based design in the Adapter layer.

