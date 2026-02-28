# Implementation Plan: IDE Deep-Linking Integration




## Overview




The implementation strategy follows a domain-first approach by defining the link-generation logic within the adapters before integrating it into the reporting infrastructure. Initially, we will implement an environment detection layer to distinguish between interactive terminals and CI/CD pipelines. This ensures we adhere to the CI Transparency Invariant early in the development cycle.

Following this, we will build a LinkProvider that leverages OSC 8 (Operating System Command) escape sequences. This provider will be responsible for transforming file paths and line numbers into clickable URIs. The final phase involves integrating this provider into the main reporting output, ensuring that users can jump directly to specific lines of code in their IDE while maintaining clean, link-free logs in non-interactive environments.




## Tasks




- [ ] F5-T1. Foundation and Environment Logic

- [ ] F5-T1.1 Implement EnvironmentDetector

- Create `src/infrastructure/env/detector.py`.

- Implement `EnvironmentDetector` class with `is_interactive()` method.

- Logic should check for `CI` environment variables and `sys.stdout.isatty()`.

- Verification: Ensure `is_interactive()` returns `False` when `CI=true` is set.

- _Requirements: 3_


- [ ] F5-T1.2 Develop OSC 8 LinkProvider

- Create `src/adapters/links/link_provider.py`.

- Implement `OSC8LinkProvider` class.

- Define `format_link(text: str, file_path: str, line: int)` method.

- Implement the OSC 8 sequence pattern: `\033]8;;file://{host}{path}#L{line}\033\\{text}\033]8;;\033\\`.

- Verification: Use unit tests to check the raw string output of the linker.

- _Requirements: 1, 2_



- [ ] F5-T2. Checkpoint: Environment and Linker Validation

- [ ] F5-T2.1 Property Test: CI Transparency Invariant

- Create property-based test `test_ci_transparency` in `tests/test_links.py`.

- Assert that for any input, if `EnvironmentDetector.is_interactive()` is false, `OSC8LinkProvider` returns the original text without escape sequences.


- [ ] F5-T2.2 Property Test: Line-Jump Precision Verification

- Create unit test `test_line_jump_format`.

- Assert that the generated URI contains the exact numeric value passed as the 'line' argument.



- [ ] F5-T3. UI Integration and Refinement

- [ ] F5-T3.1 Integrate Links into Terminal Report

- Modify `src/adapters/reporting/terminal_report.py` (referenced as E9).

- Inject `LinkProvider` into the report generation class.

- Update the file-path rendering logic to wrap paths in the generated OSC 8 sequences.

- Ensure absolute paths are resolved for the `file://` URI scheme.

- _Requirements: 1, 2_


- [ ] F5-T3.2 IDE-Specific Customization (*) *

- Implement IDE-specific detection or configuration in `OSC8LinkProvider` (e.g., custom schemes like `vscode://file/{path}:{line}`).

- Provide a fallback to the standard `file://` protocol.

- _Requirements: 1, 2_



- [ ] F5-T4. Checkpoint: Final Acceptance

- [ ] F5-T4.1 End-to-End Verification

- Run end-to-end integration tests on an interactive TTY.

- Run tests in a simulated CI environment (setting `CI=true`).

- Verify that terminal links are clickable in supported terminals (i.e., iTerm2, VS Code Terminal, Alacritty).

- _Requirements: 1, 2, 3_




## Notes




- Tasks marked with (*) are optional extensions for specific IDEs beyond standard URI patterns.

- Traceability: requirement_refs ensure every feature requirement maps to a concrete implementation step.

- Ordering Constraint: The EnvironmentDetector must be implemented before the LinkProvider to ensure the 'CI Transparency' property is not violated during integration.

- Backward Compatibility: The default behavior of the reporter must fall back to standard text formatting if terminal links are disabled or unsupported.

