# Implementation Plan: Large-Scale Repository Scanner




## Overview




The implementation strategy follows a domain-driven approach, prioritizing the file discovery and task partitioning core before developing the reporting interface. We will first establish the data models for repository trees and file metadata to support incremental scanning and parallel execution. This ensures that the foundation for 'Incremental Correctness' and 'Parallel Efficiency' is validated before the UI layer is added.

The process is divided into four main phases: establishing the incremental cache and discovery engine, implementing the parallel task dispatcher, building the TUI reporter, and finally validating the system against performance and correctness properties. This sequence ensures that we have a functional CLI tool that can be tested in headless mode before finalizing the terminal user interface.




## Tasks




- [ ] F2-T1. Domain Core & Incremental Engine

- [ ] F2-T1.1 Implement Incremental Result Cache

- Create `src/domain/models.py` to define `FileMetadata` (path, mtime, hash) and `ScanResult`.

- Implement `src/infrastructure/cache.py` using a lightweight KV store (e.g., JSON or SQLite) to persist file hashes.

- Define `should_scan(file_path)` logic based on hash comparisons.

- Verification: Unit test that `should_scan` returns False for unchanged files based on the cache.

- _Requirements: E8, E27_


- [ ] F2-T1.2 Develop Recursive Directory Discovery

- Create `src/usecases/scanner.py`.

- Implement `RecursiveDiscovery` class to walk directories and identify project roots (e.g., presence of package.json/pyproject.toml).

- Integrate with the Cache to filter out unchanged files early in the discovery phase.

- Verification: Verify discovery identifies all sub-projects in a nested test repository.

- _Requirements: E3, E6_


- [ ] F2-T1.3 Checkpoint: Incremental Logic Validation

- Verify that the Scanner correctly identifies only changed files in a multi-project setup.

- Ensure the cache is updated only after a successful file scan.



- [ ] F2-T2. Parallel Execution Framework

- [ ] F2-T2.1 Implement Task Partitioning & Execution

- Implement `TaskDispatcher` in `src/usecases/dispatcher.py`.

- Create `PartitioningStrategy` to group files into chunks for worker processes.

- Implement `ParallelExecutor` using Python's `multiprocessing` or `concurrent.futures`.

- Verification: Mock a workload and verify that CPU utilization scales with the number of workers.

- _Requirements: E8_


- [ ] F2-T2.2 Property Test: Parallel Efficiency Validation

- Write a property-based test using `Hypothesis` or a similar framework to verify `T <= (TotalTime/N) + Overhead`.

- Simulate varying file counts and sizes to ensure the dispatcher doesn't choke on small files.



- [ ] F2-T3. Reporting & UI Adapters

- [ ] F2-T3.1 Develop Aggregated TUI Reporter

- Create `src/adapters/tui_reporter.py` using `rich` or `blessed`.

- Implement `AggregatedReport` to collect results from `TaskDispatcher`.

- Format results into a project-by-project summary with pass/fail status.

- Verification: Manually trigger a scan and verify the TUI layout matches the design for organization-wide visibility.

- _Requirements: E9, E10, E7, E15_


- [ ] F2-T3.2 Property Test: Reporting Totality

- Create a test suite that checks if every project root found by `Scanner` has a corresponding row in the `TUI` output.

- Ensure 0 files are lost during the aggregation from worker processes.



- [ ] F2-T4,sub_tasks:[{description:. * Optional: Optimize cache read/writes for ultra-large repos.

- Final system integration and performance benchmarking.


- [ ] F2-T4.2. Final Checkpoint: End-to-End Integration

- Run a full scan on a repository with 50+ projects.

- Verify that only changed files are scanned on the second run.

- Confirm TUI displays all results correctly.

- _Requirements: E3, E8, E9_


- [ ] F2-T4.3. Property Test: Final Correctness Audit

- Verify zero violations of the established Correctness Properties.

- Document performance delta vs. single-threaded non-incremental scans.



## Notes




- Tasks marked with (*) are optional optimizations for very large SSDs (e.g., memory-mapped caching).

- Systematic traceability is maintained by referencing requirement IDs (e.g., E3, E8) and correctness properties.

- Ordering follows a 'Domain-first' approach to ensure the core logic of discovery and partitioning is stable before UI development.

- The 'Split-first' strategy is applied to the Scanner Engine to separate file discovery from file processing logic.

