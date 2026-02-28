# Implementation Plan: High-Performance AST Parsing Engine




## Overview




The implementation strategy follows a domain-first approach, prioritizing the High-Fidelity Parsing Worker and data models to ensure core structural analysis is robust before scaling. We will first build the parsing logic and diagnostic data structures, followed by the incremental state management to handle performance, then the parallel orchestrator to manage concurrency, and finally the terminal UI for result visualization.

This ordering is selected because the Parallel Orchestrator and UI depend heavily on the output format and performance characteristics of the Worker. By establishing the 'Detection Completeness' and 'Mapping Fidelity' properties early, we ensure that subsequent speed optimizations do not compromise the accuracy of the linting results.




## Tasks




- [ ] F0a-T1. Core Domain and Parsing Implementation

- [ ] F0a-T1.1 Define Domain Entities and Diagnostic Models

- Create `src/domain/models.py` to define `Diagnostic`, `Coordinate`, and `IssueSeverity`.

- Implement `Diagnostic` class with fields for line, column, end_line, end_column, and message.

- Ensure the model supports JSON serialization for caching.

- _Requirements: 1, 3_


- [ ] F0a-T1.2 Implement High-Fidelity Parsing Worker

- Implement `ParsingWorker` in `src/usecases/worker.py`.

- Create `ASTVisitor` subclass using the `ast` module to traverse the tree.

- Implement logic to capture precise line/column offsets for identified nodes.

- Verification: Run unit tests against a suite of Python files with known logical errors.

- _Requirements: 1, 3_



- [ ] F0a-T2. Scalability and Performance Infrastructure

- [ ] F0a-T2.1 Develop Incremental State Manager

- Create `FileCache` in `src/infrastructure/cache.py`.

- Implement content hashing (SHA-256) to track file changes.

- Implement `get_cached_diagnostics` and `save_diagnostics` methods.

- Verification: Assert that re-running the tool on unchanged files returns results without re-triggering the AST visitor.

- _Requirements: 2_


- [ ] F0a-T2.2 Implement Parallel Orchestrator

- Implement `ParallelOrchestrator` in `src/usecases/orchestrator.py`.

- Use `concurrent.futures.ProcessPoolExecutor` to distribute files to `ParsingWorker` instances.

- Integrate the `FileCache` to skip already-processed files during orchestration.

- Implement a progress reporter to track completion percentages.

- _Requirements: 2_



- [ ] F0a-T3. Verification and Testing

- [ ] F0a-T3.1 Checkpoint: Property and Performance Verification

- Run a battery of tests to ensure ‘Detection Completeness’ by verifying that a known list of vulnerabilities in a test suite are all flagged.

- Verify ‘Mapping Fidelity’ by checking that the coordinate pairs in diagnostics exactly match the source code character locations.

- Verify ‘Incremental Consistency’ by running the orchestrator twice on a large repository and ensuring the outputs are bit-for-bit identical.

- _Requirements: 1, 2_



- [ ] F0a-T4. Presentation Layer

- [ ] F0a-T4.1 Implement High-Fidelity Result Renderer

- Implement `TerminalRenderer` in `src/adapters/terminal_ui.py`.

- Use the `Coordinate` data from diagnostics to extract and highlight the specific code snippet.

- Implement ANSI color coding for different severity levels (e.g., Red for Errors, Yellow for Warnings).

- Integrate with `rich` or a similar library for advanced terminal formatting.

- _Requirements: 3_




## Notes




- Tasks marked with (*) are optional optimizations for the initial MVP.

- Domain-first ordering ensures that the core parsing logic is stable before building the infrastructure for orchestration and UI.

- Property-refs are used to ensure that the implementation directly addresses the formal correctness criteria defined in the specification.

- Backward compatibility is maintained by ensuring the Incremental State Manager's cache schema is versioned.

