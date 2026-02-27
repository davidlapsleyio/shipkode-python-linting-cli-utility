# Implementation Plan: Parallel Execution Runner Logic




## Overview




The implementation strategy follows a domain-driven approach, prioritizing the core logic of incremental analysis and result aggregation before integrating with parallel execution infrastructure. We first establish the 'Incremental Analyzer' to filter the workload, ensuring we only process what is necessary. This is followed by building a robust 'Worker Pool Manager' that abstracts the complexities of multi-core communication.

Finally, the 'Execution Orchestrator' is implemented to tie these components together, managing the lifecycle of a linting run from file discovery to final report generation. This sequence ensures that the most complex logic (parallel synchronization and resource management) is built upon stable, verified domain services.




## Tasks




- [ ] F3-T1. Domain Logic and Incremental Foundation

- [ ] F3-T1.1 Define Domain Models and Incremental Logic

- Create `src/domain/execution_types.ts` to define `FileManifest`, `AnalysisResult`, and `WorkerTask` interfaces.

- Implement `IncrementalAnalyzer` in `src/usecases/incremental_analyzer.ts`.

- Logic: Compare current file hashes against metadata cache.

- Method: `getFilesToProcess(files: string[]): Promise<string[]>`

- Method: `updateCache(results: AnalysisResult[]): void`

- _Requirements: 2_


- [ ] F3-T1.2 Implement Parallel Result Aggregation

- Implement `ResultAggregator` in `src/usecases/result_aggregator.ts`.

- Logic: Reduce an array of `AnalysisResult` into a single `ConsolidatedReport`.

- Handle deduplication of global errors and sum individual file issues.

- Verification: Unit test with mocked worker outputs to ensure sum equals total.

- _Requirements: 3_



- [ ] F3-T2. Parallel Execution Infrastructure

- [ ] F3-T2.1 Implement Multi-Core Worker Pool Manager

- Implement `WorkerPoolManager` in `src/adapters/worker_pool.ts`.

- Use Node.js `worker_threads` or `child_process` modules.

- Logic: Determine worker count using `os.cpus().length - 1`.

- Method: `setupPool()` and `dispatchTask(file: string): Promise<AnalysisResult>`.

- _Requirements: 1_


- [ ] F3-T2.2 Develop Worker Thread Script and Messaging

- Create `src/adapters/worker_entry.ts` as the entry point for worker threads.

- Implement message passing protocol for sending file paths and receiving results.

- Ensure error handling for worker crashes.

- _Requirements: 1_



- [ ] F3-T3. Integration Checkpoint I

- [ ] F3-T3.1 Checkpoint: Resource and Logic Verification

- Execute unit tests for `WorkerPool` allocation logic.

- Verify that on an N-core machine, exactly N-1 workers are spawned.

- Verify `IncrementalAnalyzer` correctly excludes files with matching hashes.



- [ ] F3-T4. Orchestration and Reliability

- [ ] F3-T4.1 Implement Execution Orchestrator

- Implement `ExecutionOrchestrator` in `src/usecases/execution_orchestrator.ts`.

- Logic:

- 1. Call IncrementalAnalyzer to prune file list.

- 2. Batch files and dispatch to WorkerPoolManager.

- 3. Collect results and pass to ResultAggregator.

- 4. Return final report to the UI/CLI layer.

- _Requirements: 1, 2, 3_


- [ ] F3-T4.2 Property-Based Testing Suite

- Create `tests/properties/execution_engine.test.ts`.

- Test 'Strict Incrementalism': Run engine twice, verify second run executes zero tasks.

- Test 'Conservation of Results': Mock 100 workers with random issue counts, verify aggregator sum.

- Test 'Resource Boundary Adherence': Spy on process spawning to ensure limit compliance.



- [ ] F3-T5. Final Checkpoint

- [ ] F3-T5.1 Final System Integration Test

- Run the full suite against a large codebase (>1000 files).

- Validate PR scan time is < 60 seconds as per requirement.

- Inspect the final report for consolidated health metrics.

- _Requirements: 1, 3_




## Notes




- Tasks marked with (*) are optional optimizations for very large repositories.

- Traceability: requirements and properties are referenced to ensure all acceptance criteria are met.

- Ordering Constraint: The Worker Pool Manager must be implemented before the Orchestrator to ensure the execution pipeline has a valid target for distribution.

- Compatibility: The hash-based cache must be backward compatible with older scan metadata if present.

