# Implementation Plan: Large-Scale Repository Scanner




## Overview




The implementation strategy follows a domain-driven approach, starting with the core scanning logic and task partitioning models before moving toward infrastructure-specific parallel execution. We prioritize the 'Task Partitioner' and 'Incremental Tracker' first because the efficiency of the large-scale scanner depends on the ability to minimize work (delta tracking) and distribute it correctly.

Once the domain logic for partitioning is stable, we implement the hardware-specific worker pool and the file system adapters. This ensures that the parallel execution logic is decoupled from the file discovery logic. The final phase integrates these into a Scan Orchestrator that enforces quality gates and security invariants, providing a unified entry point for both local development and CI/CD pipelines.




## Tasks




- [ ] F2-T1. Domain Logic & Partitioning

- [ ] F2-T1.1 Implement Task Partitioning Logic

- Create `src/domain/models.py` to hold `ScanTask` and `FileMetadata` dataclasses.

- Implement `src/usecases/partitioner.py` with `TaskPartitioner` class.

- Add logic to split a list of file paths into N roughly equal batches.

- Verification: Unit test with varying file counts and core counts to ensure even distribution.

- _Requirements: 2_


- [ ] F2-T1.2 Implement Incremental Delta Tracking

- Create `src/domain/delta_tracker.py` to manage file hashes and timestamps.

- Implement logic to compare current file state against a persistent `.scan_cache` file.

- Implement the 'Incremental Completeness Invariant' check logic.

- Verification: Mock file system changes and ensure only 'modified' files are returned in the task list.

- _Requirements: 3_



- [ ] F2-T2. Checkpoint: Domain Logic Validation

- [ ] F2-T2.1 Checkpoint: Domain Validation

- Run suite of unit tests for `TaskPartitioner` and `DeltaTracker`.

- Verify that 0 files are scanned when no changes occur.

- Verify that all files are scanned on the first run.



- [ ] F2-T3. Adapters & Infrastructure

- [ ] F2-T3.1 Recursive File Discovery Adapter

- Implement `FileSystemAdapter` in `src/adapters/filesystem.py`.

- Add `recursive_search(pattern: str)` method using `pathlib.rglob`.

- Ensure it ignores directories specified in a `.scannerignore` file.

- Verification: Test against a nested directory structure with 1000+ dummy files.

- _Requirements: 1_


- [ ] F2-T3.2 Parallel Execution Infrastructure

- Implement `WorkerPool` in `src/infrastructure/worker_pool.py` using `multiprocessing.Pool`.

- Implement `map_parallel(tasks, func)` wrapper with error handling.

- Verification: Use a sleep-heavy mock task to verify that P cores are utilized simultaneously.

- _Requirements: 2_



- [ ] F2-T4. Orchestration & Integration

- [ ] F2-T4.1 Orchestrator & Quality Gates

- Implement `ScanOrchestrator` in `src/usecases/orchestrator.py`.

- Coordinate Phase 1 logic (Delta Tracking) with Phase 3 logic (Parallel Execution).

- Implement the 'Security Gate Failure Invariant' logic: check results for 'Critical' tags and raise `SecurityGateViolation`.

- Verification: Run integrated scan on a test repo with known 'Critical' dummy vulnerabilities.

- _Requirements: 4_



- [ ] F2-T5. Final Verification

- [ ] F2-T5.1 Final Property & Integration Testing

- Execute `tests/property/test_parallelism.py` to validate 'Maximum Parallelism Invariant'.

- Execute `tests/property/test_incremental.py` to validate 'Incremental Completeness Invariant'.

- Validate CI pipeline exit codes (0 for pass, non-zero for 'Critical' finds).




## Notes




- Tasks marked with (*) in descriptions are optional performance optimizations.

- Requirement traceability is maintained via requirement_refs to ensure every user story is addressed.

- Ordering follows a 'Domain-First' approach: core logic and partitioning are built before infrastructure-specific worker pools.

- Compatibility: The FileSystem adapter must maintain POSIX compliance for containerized CI environments.

