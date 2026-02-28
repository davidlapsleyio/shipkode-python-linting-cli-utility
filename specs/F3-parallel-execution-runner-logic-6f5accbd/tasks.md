# Implementation Plan: Parallel Execution Runner Logic




## Overview




The implementation follows a domain-driven approach, starting with the core 'Job' entities and incremental logic before moving to the orchestration layer. We prioritize the 'Job' domain and 'Cache Controller' first to ensure that the core logic for what needs to run is decoupled from how it runs (parallel vs. sequential). 

The second phase introduces the Task Scheduler and Parallel Runner Orchestrator using a process pool strategy to bypass the Global Interpreter Lock (GIL) and maximize multi-core utilization. Verification checkpoints and property-based tests are integrated after each phase to ensure deterministic output and resource constraints are strictly followed.




## Tasks




- [ ] F3-T1. Domain and Cache Layer Implementation

- [ ] F3-T1.1 Define Lint Job Domain Entities

- Create/Update `src/domain/job.py`.

- Define `LintJob` dataclass with fields: `file_path`, `ruleset_hash`, `content_hash`, and `priority`.

- Implement `__eq__` and `__hash__` to support set operations for job comparison.

- Verification: Unit tests for `LintJob` equality and hashing.

- _Requirements: F3.1, F3.2_


- [ ] F3-T1.2 Implement Incremental Cache Controller

- Create `src/adapters/cache_controller.py`.

- Implement `CacheController` class with `is_cached(file_path, ruleset_hash)` and `update_cache(file_path, ruleset_hash, results)`.

- Use a persistent storage backend (e.g., JSON or SQLite) to store file hashes and timestamps.

- Verification: Ensure cached files return 'True' only if content and ruleset haven't changed.

- _Requirements: F3.2_


- [ ] F3-T1.3 Checkpoint: Cache Logic Verification

- Execute unit tests for CacheController.

- Verify that a mocked rule-runner produces identical outputs when reading from cache vs. fresh execution.

- _Requirements: F3.2_



- [ ] F3-T2. Parallel Execution Engine

- [ ] F3-T2.1 Implement Task Scheduler Batching

- Create `src/usecases/scheduler.py`.

- Implement `TaskScheduler` to group `LintJob` items into batches based on file size or rule complexity.

- Implement a `get_next_batch()` method that respects `N` available cores.

- Verification: Test batching logic ensures no jobs are dropped or duplicated.

- _Requirements: F3.1_


- [ ] F3-T2.2 Implement Parallel Runner Orchestrator Logic

- Create `src/usecases/parallel_runner.py`.

- Implement `ParallelRunnerOrchestrator` using `multiprocessing.Pool` or `concurrent.futures.ProcessPoolExecutor`.

- Implement worker function that takes a `LintJob`, executes rules, and returns results.

- Ensure the orchestrator handles keyboard interrupts and worker failures gracefully.

- Verification: Run parallel execution on a test suite and compare results to sequential output.

- _Requirements: F3.1, F3.3_


- [ ] F3-T2.3 Property Test: Execution Determinism

- Use a property-based testing tool (e.g., Hypothesis) to verify 'Execution Determinism'.

- Validate that ParallelRunner(F) == SequentialRunner(F) for randomized file sets and rule counts.

- _Requirements: F3.1_



- [ ] F3-T3. Integration and CI/CD Readiness

- [ ] F3-T3.1 Resource Management and CLI Integration

- Add logic to auto-detect CPU count using `os.cpu_count()`.

- Add a `--parallel` and `--jobs N` flag to the CLI entry point.

- Implement resource limiting to ensure the runner never spawns more than N+1 processes.

- Verification: Use `psutil` in tests to monitor process count during execution.

- _Requirements: F3.1, F3.3_


- [ ] F3-T3.2 CI/CD Performance Integration Test

- Create a smoke test script that mimics a CI environment (restricted memory/CPU).

- Run the ParallelRunner on a repository with 1000+ files and verify performance gain vs sequential mode.

- Verification: CI pipeline terminates successfully with non-zero exit code if violations are found.

- _Requirements: F3.3_


- [ ] F3-T3.3 Checkpoint: Final Integration & Cache Integrity Verification

- Final end-to-end check.

- Verify 'Incremental Cache Integrity' by running the linter twice on the same codebase and ensuring 0 rules are executed on the second pass.

- _Requirements: F3.2_




## Notes




- Tasks marked with (*) are optional optimizations but recommended for high-performance CI. Traceability is maintained via requirement_refs to ensure every feature is accounted for in the implementation. A 'split-first' strategy is used for the domain entities to ensure clean interfaces before concurrency is introduced. Backward compatibility is guaranteed by ensuring the ParallelRunner implements the same interface as the existing sequential logic.

