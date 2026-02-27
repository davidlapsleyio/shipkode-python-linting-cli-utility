# Implementation Plan: Comprehensive Python Rule Suite




## Overview




The implementation follows a domain-driven approach, starting with the core data models and rule definitions (Domain), followed by the logic for running these rules (Use Cases), the integration with external scanning tools (Adapters), and finally the presentation layer (Rich TUI). This sequence ensures that the engine can process results independently of how they are gathered or displayed.

The strategy prioritizes the 'Security' and 'Logical' detection capabilities early in the development cycle to satisfy high-risk requirements before polishing the aesthetic reporting. By decoupling the Rule Execution Engine from the External Linter Bridge, we allow for mocking external tool outputs during initial development and testing.




## Tasks




- [ ] F1-T1. Phase 1: Domain Models and Core Interfaces

- [ ] F1-T1.1 Define Domain Models and Enums

- Create `src/domain/enums.py` containing `ViolationCategory` (STYLE, LOGIC, SECURITY) and `Severity` (LOW, MEDIUM, HIGH, CRITICAL).

- Define `Violation` dataclass in `src/domain/models.py` with fields: `id`, `category`, `message`, `line_number`, `column`, and `file_path`.

- Ensure `Severity` implements comparison logic to support prioritization.

- _Requirements: 4_


- [ ] F1-T1.2 Define Adapter Interfaces

- Create `src/domain/interfaces.py`.

- Define `AbstractLinter` interface with an `execute(source_path: Path) -> List[Violation]` method.

- Define `ReportGenerator` interface for the TUI output.

- _Requirements: 1, 2, 3_



- [ ] F1-T2. Phase 2: Rule Engines and Use Cases

- [ ] F1-T2.1 Implement Rule Execution Engine

- Implement `RuleExecutionEngine` class.

- Add `run_all(paths: List[Path])` method that iterates through registered adapters.

- Implement error handling for file access and external process timeouts.

- _Requirements: 2_


- [ ] F1-T2.2 Implement Report Aggregator with Prioritization

- Implement `ReportAggregator` in `src/usecases/report_aggregator.py`.

- Add logic to group violations by `ViolationCategory`.

- Implement sorting logic where SECURITY and LOGIC violations always precede STYLE.

- _Requirements: 4_



- [ ] F1-T3. Phase 3: External Linter Adapters

- [ ] F1-T3.1 Implement PEP 8 Bridge

- Implement `ExternalLinterBridge` in `src/adapters/linter_bridge.py`.

- Integrate `flake8` or `pycodestyle` for PEP 8 compliance.

- Map external error codes to `ViolationCategory.STYLE`.

- _Requirements: 1_


- [ ] F1-T3.2 Implement Security and Logic Bridge

- Integrate `bandit` or `safety` within the bridge for security scanning.

- Map findings like `eval()` usage to `ViolationCategory.SECURITY`.

- Integrate `pyflakes` or `mypy` (static mode) for logic error detection.

- _Requirements: 2Attribute, 3_



- [ ] F1-T4. Phase 4: Checkpoint & Property Verification

- [ ] F1-T4.1 Property Test: Severity Prioritization

- Create `test_prioritization.py` to verify that SECURITY items appear at the top of lists regardless of discovery order.


- [ ] F1-T4.2 Property Test: Security Detection Coverage

- Create `test_security_coverage.py` using a suite of 'vulnerable' python scripts.

- Assert that `eval()` and `subprocess.shell=True` trigger violations.


- [ ] F1-T4.3 Property Test: PEP 8 Enforcement

- Create `test_style_compliance.py` with misindented code.

- Assert that the Bridge correctly identifies and reports these as STYLE violations.



- [ ] F1-T5. Phase 5: Presentation Layer

- [ ] F1-T5.1 Implement Rich TUI Reporting

- Use `rich.console` and `rich.table` to build the UI.

- Color code categories: Red for Security, Yellow for Logic, Blue for Style.

- Display a summary table showing count of violations per category.

- _Requirements: 4_




## Notes




- Tasks marked with (*) are optional optimizations for the Rich UI.

- Traceability: Requirement IDs are linked to sub-tasks to ensure every user story and edge case is accounted for.

- Ordering Constraints: Data models must be established before the Bridge can map external outputs to internal types.

- Compatibility: The Linter Bridge maintains a generic interface so that underlying tools (flake8, bandit) can be swapped without affecting the Rule Engine.

