# Implementation Plan: Automated Quality Gate Logic




## Overview




The implementation strategy follows a domain-driven approach, starting with the core gate logic and models before moving to external reporting and CLI integration. We first define the 'QualityGate' domain model and the 'QualityEvaluator' service to establish the business logic for 'failure' independently of how it is reported or handled by the OS. This ensures the core logic is testable without side effects.

Once the evaluation logic is solid, we implement the 'Machine-Readable Exporters' to handle data persistence and dashboard integration. Finally, we wire these components into the 'CLI Termination Controller', which acts as the infrastructure layer responsible for converting domain 'failures' into system exit codes. This order ensures that the most critical logic (deciding if a build is safe) is prioritized and verified before addressing delivery formats or process signals.




## Tasks




- [ ] F5-T1. Domain Logic: Quality Gate Evaluation

- [ ] F5-T1.1 Define Quality Gate Domain Models

- Create `src/domain/quality_gate.py` to hold the `GateRule` and `QualityStatus` data classes.

- Define `Severity` enum (INFO, LOW, MEDIUM, HIGH, CRITICAL).

- Define `GateConfiguration` to hold `fail_on_severity` and `custom_exit_code` mappings.

- _Requirements: 3_


- [ ] F5-T1.2 Implement QualityEvaluator Service

- Create `src/usecases/quality_evaluator.py`.

- Implement `QualityEvaluator.evaluate(violations, config)` logic.

- Logic must iterate through violations and set `is_failed=True` if any violation severity >= `config.fail_on_severity`.

- Return a `QualitySummary` object containing the failure status and counts by severity.

- _Requirements: 3_


- [ ] F5-T1.3 Property Test: Threshold Enforcement

- Create `tests/properties/test_gate_properties.py`.

- Use Hypothesis to generate `AnalysisResult` objects with varying severity levels.

- Assert that the evaluator consistently marks results as failed when the threshold is met or exceeded.



- [ ] F5-T2. Adapters: Machine-Readable Reporting

- [ ] F5-T2.1 Implement JSON Report Formatter

- Create `src/adapters/report_exporter.py`.

- Implement `JSONFormatter` class to serialize the `AnalysisResult` and `QualitySummary` to a standard JSON object.

- Ensure all fields from the domain model are mapped.

- _Requirements: 2_


- [ ] F5-T2.2 Implement SARIF 2.1.0 Formatter

- Implement `SarifFormatter` class in `src/adapters/report_exporter.py`.

- Map internal violations to SARIF `runs`, `results`, and `rules` objects.

- Ensure `level` mapping (e.g., CRITICAL -> 'error', LOW -> 'note').

- _Requirements: 2_


- [ ] F5-T2.3 Property Test: SARIF Schema Compliance

- Create `tests/integration/test_sarif_compliance.py`.

- Instantiate `SarifFormatter` with sample data.

- Validate the output against the official SARIF 2.1.0 JSON schema using `jsonschema` library.

- _Requirements: 2_



- [ ] F5-T3. Intermediate Checkpoint

- [ ] F5-T3.1 Checkpoint: Evaluation & Export Logic

- Execute all unit tests for `QualityEvaluator` and `ReportExporter`.

- Verify coverage for all severity edge cases (at threshold, above, below).



- [ ] F5-T4. Infrastructure: CLI & Process Control

- [ ] F5-T4.1 Update CLI Controller with Evaluation Flow

- Update `src/infrastructure/cli_controller.py`.

- Integrate the `QualityEvaluator` into the main execution flow.

- Ensure that if `evaluator.evaluate()` returns a failure status, the controller retrieves the configured `exit_code`.

- _Requirements: 1_


- [ ] F5-T4.2 Implement Exit Code Enforcement

- Implement the final `sys.exit(code)` call in the `cli_controller`.

- Ensure that this is the last operation after all reports (JSON/SARIF) have been written to disk.

- _Requirements: 1_


- [ ] F5-T4.3 Property Test: Termination Consistency

- Create `tests/integration/test_cli_termination.py`.

- Use `subprocess` or `py.test`'s `capsys` to run the CLI with a mock scan result that triggers a failure.

- Assert that the exit code matches the defined threshold configuration.

- _Requirements: 1_



- [ ] F5-T5. Final Verification

- [ ] F5-T5.1 Final Checkpoint: E2E Quality Gate Integration

- Run a full scan against a known "vulnerable" target.

- Verify that a SARIF file is produced, it is valid, and the CLI returns a non-zero exit code.

- Repeat with a "clean" target and verify a 0 exit code.

- _Requirements: 1 status="ready", 2 status="ready", 3 status="ready"_




## Notes




- Tasks marked with (*) are optional optimizations for the dev environment and not strictly required for MVP.

- Traceability: requirement_refs ensure that every functional need from the user story is mapped to a specific implementation step.

- Ordering Constraints: Domain models for gates must be defined first to establish the 'contract' between the evaluator and the CLI.

- Compatibility: The SARIF exporter uses the 2.1.0 standard to ensure compatibility with GitHub Advanced Security and other industry dashboards.

