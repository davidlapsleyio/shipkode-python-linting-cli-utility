# Implementation Plan: Rich Visual Terminal UI




## Overview




The implementation strategy follows a domain-first approach, starting with the health aggregation logic to ensure statistical accuracy before any visual work begins. By building the 'Repository Health Aggregator' first, we establish a single source of truth for the data displayed in the UI.

The second phase focuses on atomic UI components: the syntax highlighter for code snippets and the data-table formatting. Once these components are verified against correctness properties (accuracy and resilience), the final phase integrates them into a cohesive Terminal UI Adapter. This incremental approach allows us to test reporting logic in isolation from terminal-specific rendering issues.




## Tasks




- [ ] F2-T1. Domain Logic & Health Aggregation

- [ ] F2-T1.1 Implement Health Aggregator and Data Models

- Create src/domain/health_aggregator.py.

- Implement HealthSummary dataclass to hold categorized counts.

- Implement HealthAggregator class with aggregate_results(results: List[LintResult]) method.

- Logic must group violations by severity and rule type.

- _Requirements: R3_


- [ ] F2-T1.2 Verify Statistical Integrity Property

- Create tests/domain/test_health_aggregator.py.

- Write unit tests verifying that the sum of categorized errors matches the input list size.

- Ensure edge cases (empty results, multiple severities) are handled.

- _Requirements: R3_



- [ ] F2-T2. Visual Component Development

- [ ] F2-T2.1 Build Syntax Highlighting Adapter

- Create src/adapters/syntax_highlighter.py.

- Implement SyntaxHighlighter class using Rich's 'Syntax' and 'Panel' objects.

- Method highlight_snippet(path, line_no, context) must locate and highlight the specific violation line.

- Ensure line padding and line number gutter are rendered.

- _Requirements: R2_


- [ ] F2-T2.2 Property Test: Visual Context and Resilience

- Create test scripts to render snippets at various terminal widths (80, 100, 120).

- Verify that line numbers in the Rich Panel match the underlying source file indexes exactly.

- _Requirements: R2_



- [ ] F2-T3. TUI Integration and Layout Validation

- [ ] F2-T3.1 Implement TUI Adapter & Layout

- Create src/adapters/tui_adapter.py.

- Implement RichTUIAdapter class using Rich.console.Console.

- Create render_report(results: List[LintResult]) method to coordinate the Health Table and Syntax Panels.

- Use Rich 'Table' for the Health Summary with color-coded severity levels (Red for Error, Yellow for Warning).

- _Requirements: R1/R3_


- [ ] F2-T3.2 Property Test: Layout Resilience (80 chars)

- Implement property-based tests using 'Console(width=80)' to ensure no character corruption or layout breaking occurs in restricted environments.

- Verify color contrast ratios for accessibility [E10].

- _Requirements: R1_



- [ ] F2-T4. Checkpoint: Integrated Verification

- [ ] F2-T4.1 End-to-End Visual Checkpoint

- Execute a full linting run on a known codebase.

- Manually verify that the 'Categorized Health Table' counts match the 'Syntax Fragments' displayed.

- Check that the UI remains responsive and readable in different terminal emulators (e.g., iTerm2, VS Code Terminal).

- _Requirements: R1/R2/R3_




## Notes




- Optional tasks are marked with (*).

- Requirement traceability (e.g., R1, R2) ensures every feature goal is mapped to a specific engineering action.

- Property references (e.g., P1, P2) ensure that validation logic is built into the development lifecycle.

- Backward compatibility: The TUI adapter will coexist with existing CLI outputters via a common interface, ensuring no breaking changes for CI/CD environments preferring raw logs.

- Ordering Constraints: Highlighting components (2.2) must be completed before the main TUI adapter (3.1) to allow for integrated visual testing.

