# Implementation Plan: Automated Quality Gate Logic




## Overview




This plan implements a robust 'Quality Gate' system by first defining the domain models for thresholds and evaluation logic, then building the reporting infrastructure, and finally integrating these into the CLI. We prioritize the 'Quality Gate Evaluator' domain service to ensure core logic is independent of the delivery mechanism (CLI or JSON).

The strategy follows a split-first approach, ensuring that the reporting logic is isolated from the CLI orchestration. We use a 'Domain-Driven' sequence where the gate logic is built and verified with unit tests before the CLI is modified to support non-zero exit codes. This ensures that the 'Negative Gate Integrity' property is maintained throughout the development lifecycle.




## Tasks




- [ ] F5-T1. Domain Logic and Gate Models

- [ ] F5-T1.1 Define Domain Gate Models

- Create `src/domain/gate_models.py`.

- Implement `ThresholdConfig` dataclass with fields for 'critical', 'high', 'medium', and 'low'.

- Implement `GateResult` dataclass to store the pass/fail status and a summary of violations.

- Verification: Unit test that `ThresholdConfig` can be instantiated from a dictionary.

- _Requirements: 3_


- [ ] F5-T1.2 Implement Gate Evaluator Service

- Create `src/usecases/gate_evaluator.py`.

- Implement `QualityGateEvaluator` class.

- Add method `evaluate(findings: List[Finding], config: ThresholdConfig) -> GateResult`.

- Logic must aggregate counts per severity and compare against thresholds.

- Verification: Unit tests with mock findings to ensure 'pass' on zero findings and 'fail' on threshold overflow.

- _Requirements: 3_



- [ ] F5-T2. Reporting and Format Adapters

- [ ] F5-T2.1 Implement Machine-Readable Exporters

- Create `src/adapters/report_exporter.py`.

- Implement `JsonExporter` class to output the standard internal report format.

- Implement `SarifExporter` class using a template or library to map internal `Finding` objects to SARIF v2.1.0 JSON schema.

- Verification: Run generated SARIF outputs through an online JSON schema validator against the official v2.1.0 schema.

- _Requirements: 2_


- [ ] F5-T2.2 Integrate Gate Status into Exporters

- Ensure `GateResult` status ('pass'/'fail') is included in the JSON/SARIF output metadata.

- Verification: Assert that the 'pass' status is correctly recorded in the JSON output when thresholds are met.

- _Requirements: 2_



- [ ] F5-T3. Checkpoint: Internal Logic Validation

- [ ] F5-T3.1 Run Unit Tests for Gate Logic and Exporters

- Execute `pytest tests/unit/gate_evaluator_test.py`.

- Execute `pytest tests/unit/exporter_test.py`.

- Verify 100% coverage on gate logic.



- [ ] F5-T4. CLI Integration and Orchestration

- [ ] F5-T4.1 Update CLI Command Plane

- Modify `src/infrastructure/cli.py` to add `--thresholds` (kv-pair), `--format` (json/sarif), and `--output-file` flags.

- Use `argparse` or `click` to parse these into `ThresholdConfig`.

- Verification: Run `pyvisualguard --help` to confirm new flags exist.

- _Requirements: 2, 3_


- [ ] F5-T4.2 Implement Exit Code Orchestration

- Update `src/infrastructure/cli.py` to invoke `QualityGateEvaluator` after findings are generated.

- Call `sys.exit(1)` if `GateResult.is_failed` is true; otherwise `sys.exit(0)`.

- Verification: Execute CLI against a known 'insecure' test directory and confirm `$?` is 1.

- _Requirements: 1_



- [ ] F5-T5. Final Checkpoint: Property Validation

- [ ] F5-T5.1 Property-Based Verification

- Create `tests/property/test_gate_properties.py`.

- Test 'Threshold Violation Exit Code Consistency' by sweeping across finding counts.

- Test 'Negative Gate Integrity' by running scans against an empty directory.

- Test 'SARIF Schema Compliance' by validating actual CLI-generated files.




## Notes




- Tasks marked with (*) are optional optimizations for the CLI user experience.

- Requirement references (e.g., [1], [2]) ensure every feature requested by the DevOps Lead is mapped to a code change.

- The 'Domain-First' ordering ensures that the logic for 'what' constitutes a failure is decoupled from 'how' it is reported (CLI vs JSON).

- Compatibility: The SARIF exporter must maintain compatibility with GitHub Advanced Security and other standard SARIF consumers.

