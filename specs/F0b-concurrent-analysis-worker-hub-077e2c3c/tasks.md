# Implementation Plan: Concurrent Analysis Worker Hub




## Overview




The implementation strategy follows a domain-driven approach, prioritizing the definition of result aggregation models and the orchestration logic before building the platform-specific multiprocessing adapters. This ensures that the core business logic—how files are distributed and results merged—is decoupled from the underlying concurrency mechanism.

Initial phases focus on the Parallel Execution Orchestrator to define the scheduling contract, followed by the development of the Multiprocessing Pool Adapter to interface with system resources. The final phases integrate real-time progress monitoring and comprehensive property-based testing to ensure that parallel execution remains deterministic and respects hardware constraints.




## Tasks




- [ ] F0b-T1. Domain Logic and Orchestration Foundation

- [ ] F0b-T1.1 Define Orchestration Domain and Result Aggregation Logic

- Define 'AnalysisTask' and 'AnalysisResult' dataclasses in a new core domain module.

- Create 'ParallelOrchestrator' class in 'src/usecases/parallel_orchestrator.py'.

- Implement 'run_parallel_scan' method signature accepting a list of file paths and a worker function.

- Implement the 'reduce' logic to merge 'AnalysisResult' objects.

- Verification: Unit test result aggregation logic with a mock worker function.

- _Requirements: 3_


- [ ] F0b-T1.2 Implement Progress Tracking Data Contract

- Define 'ProgressUpdate' message schema.

- Create 'ProgressMonitor' in 'src/infrastructure/progress_monitor.py' using a thread-safe queue.

- Implement 'update()' and 'finalize()' methods for the monitor.

- Verification: Assert that the monitor correctly counts calls in a multi-threaded context.

- _Requirements: 4_



- [ ] F0b-T2. Infrastructure and Adapter Implementation

- [ ] F0b-T2.1 Develop Multiprocessing Pool Adapter

- Create 'MPPoolAdapter' in 'src/adapters/concurrency/mp_pool_adapter.py'.

- Implement 'execute_pool' using 'multiprocessing.Pool' or 'ProcessPoolExecutor'.

- Add logic to detect CPU core count using 'os.cpu_count()'.

- Implement a 'chunking' strategy to divide large file lists into batches for workers.

- Verification: Test that 'MPPoolAdapter' can execute a simple math function across multiple processes.

- _Requirements: 1, 2_


- [ ] F0b-T2.2 Integrate Adapter and Orchestrator

- Integrate 'MPPoolAdapter' into 'ParallelOrchestrator'.

- Implement error handling for worker process crashes (zombies/timeouts).

- Connect 'ProgressMonitor' to the worker callback loop to send signals back to the main process.

- Verification: Perform a manual run on a directory with 10+ files and verify all are processed.

- _Requirements: 2, 4_



- [ ] F0b-T3. Verification and Property Testing

- [ ] F0b-T3.1 Checkpoint: End-to-End Execution Flow

- Ensure system terminates cleanly after report generation.

- Check that progress bar reaches 100% and matches result count.


- [ ] F0b-T3.2 Property-Based Correctness Testing

- Create 'tests/property/test_concurrency.py'.

- Implement 'test_deterministic_aggregation': Compare output of ParallelOrchestrator vs a standard 'map()' function on 1000 dummy files.

- Implement 'test_cpu_boundary_constraint': Use 'psutil' to monitor child process count during execution, asserting it never exceeds N+1.

- Implement 'test_total_task_completion_invariant': Verify that the number of signals received by 'ProgressMonitor' exactly matches the input list length.




## Notes




- Tasks marked with (*) are optional optimizations for very large repositories.

- Traceability: Requirement references (e.g., E8, E12) ensure all stakeholder needs are mapped to code.

- Ordering Constraint: The implementation follows a 'Core-Out' approach, ensuring the data contracts for parallel results are defined before the orchestration logic or UI-based progress bars.

- Backward Compatibility: The orchestrator will fall back to a sequential execution strategy if the system environment restricts process spawning.

