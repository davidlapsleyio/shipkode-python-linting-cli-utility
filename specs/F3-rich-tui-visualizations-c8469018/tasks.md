# Implementation Plan: Rich TUI Visualizations




## Overview




The implementation follows a domain-driven approach, starting with the Violation Aggregator use-case to prepare the data structures required for visualization. By decoupling the data transformation logic from the UI rendering, we ensure that the summary counts remain consistent regardless of how they are styled. 

Once the data layer is solid, we move to the Theme Manager and individual visual components (Table and Snippet). This phase focuses on the 'Rich' library integration to fulfill the visual prominence and syntax highlighting requirements. The final phase integrates these components into a non-interactive TUI Controller that adheres to static output constraints, ensuring the terminal buffer remains scrollable as requested.




## Tasks




- [ ] F3-T1. Domain Logic & Aggregation Phase

- [ ] F3-T1.1 Implement Violation Aggregator Logic

- Create `ViolationAggregator` class in `src/usecases/violation_aggregator.py`.

- Implement `get_categorized_counts(violations)` returning a typed dictionary of Category -> Severity -> Count.

- Implement `get_grouped_by_file(violations)` to facilitate snippet rendering.

- Unit test with edge cases (empty lists, singular violations).

- _Requirements: F3.1, F3.3_


- [ ] F3-T1.2 Verify Violation Conservation Property

- Create property test `test_conservation_of_violations`.

- Input: Randomly generated lists of Violation entities using Hypothesis or similar.

- Assertion: Sum of entries in Aggregator output equals length of input list.



- [ ] F3-T2. Component & Formatting Phase

- [ ] F3-T2.1 Implement Severity-Based Theme Manager

- Implement `ThemeManager` in `src/adapters/tui/theme.py`.

- Define a `StyleSheet` class mapping `Severity` levels to ANSI colors (e.g., Critical: 'bold red', Error: 'bright_red').

- Verification: Print a color swatch to verify theme readability on dark/light backgrounds.

- _Requirements: F3.3_


- [ ] F3-T2.2 Create Summary Table Component

- Create `SummaryTableGenerator` in `src/adapters/tui/summary_table.py`.

- Use `rich.table.Table` to render the aggregated data from Phase 1.

- Apply colors from `ThemeManager` to the 'Severity' column.

- _Requirements: F3.1, F3.3_


- [ ] F3-T2.3 Develop Syntax Highlighted Snippets

- Create `SyntaxSnippetFormatter` in `src/adapters/tui/snippet.py`.

- Use `rich.syntax.Syntax` to load file segments.

- Calculate window bounds: [violation.line - 2, violation.line + 2] to center context.

- Highlight the specific error line using Rich's `highlight_lines` parameter.

- _Requirements: F3.2_



- [ ] F3-T3. Integration & Validation Phase

- [ ] F3-T3.1 Integrate TUI Controller

- Create `TUIController` in `src/adapters/tui/controller.py`.

- Implement `render_report(violations)` which sequentially prints the Table followed by Snippets.

- CONSTRAINT: Must use `rich.console.Console(force_terminal=True)` and avoid `console.alternate_screen()`.

- _Requirements: F3.4_


- [ ] F3-T3.2 Property Test: Static Output Invariant

- Test script that captures stdout.

- Verify total absence of escape sequences like `\e[?1049h` (enter alternate buffer) or mouse tracking.

- Ensure terminal scrollback is preserved after execution.


- [ ] F3-T3.3 Property Test: Severity Prominence

- Create `test_severity_visual_prominence`.

- Render a CRITICAL violation to a virtual string console.

- Regex check for standard ANSI 'bold red' codes or the specific hex/name defined in the Theme Manager.



- [ ] F3-T4. Checkpoint: Final Acceptance

- [ ] F3-T4.1 End-to-End Visual Checkpoint

- Execute full report generation against the codebase itself.

- Verify counts match expected tool output.

- Manual check: Scroll through terminal buffer to verify layout is static and formatted correctly.




## Notes




- Tasks marked with (*) are optional aesthetic enhancements.

- Requirement traceability is maintained via IDs (e.g., F3.1) to ensure every feature request has a corresponding code path.

- The 'Split-First' strategy is applied to the adapter layer to prevent the TUI controller from becoming a 'God Object'.

- Compatibility: All ANSI output remains compatible with standard xterm-256color terminals without requiring GPU acceleration.

