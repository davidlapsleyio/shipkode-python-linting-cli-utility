# Implementation Plan: Rich Visual Terminal UI




## Overview




The implementation strategy follows a domain-to-adapter flow, starting with the refinement of data models to support rich metadata, followed by the development of discrete UI components using the Rich library. We will first build the granular adapters for syntax highlighting and table construction before integrating them into a master Emitter class. This ensures that the complex visual logic is unit-testable in isolation.

The rationale for this order is to establish the 'Snippet Fidelity' and 'Completeness of Summary Aggregation' properties at the component level before shipping the final integrated UI. Later phases focus on CI/CD environment detection to satisfy the 'CI Transparency' property, ensuring the tool degrades gracefully in non-interactive shells.




## Tasks




- [ ] F2-T1. Domain Model Refinement

- [ ] F2-T1.1 Enhance Domain Models for UI Metadata

- Update `LintResult` in `src/domain/models.py` to include `context_lines` or raw source snippets if not already present.

- Define a `Severity` Enum (Error, Warning, Info) to facilitate grouping.

- Verification: Ensure domain objects can be instantiated with file coordinates.

- _Requirements: F2.1_



- [ ] F2-T2. Component Adapter Development

- [ ] F2-T2.1 Implement Syntax Highlighter Adapter

- Create `src/adapters/ui/syntax_prettifier.py`.

- Implement `SyntaxHighlighter` class using `rich.syntax.Syntax`.

- Logic: Read file at `[line_start, line_end]`, apply language-specific lexing.

- Verification: Property test 'Snippet Fidelity' by comparing rendered text against source file content.

- _Requirements: F2.2_


- [ ] F2-T2.2 Implement Summary Table Builder

- Create `src/adapters/ui/table_builder.py`.

- Implement `SummaryTableBuilder` class using `rich.table.Table`.

- Logic: Group a list of `LintResult` objects by severity and calculate totals.

- Verification: Property test 'Completeness of Summary Aggregation' with a mock list of results.

- _Requirements: F2.3_



- [ ] F2-T3. Checkpoint 1: Component Validation

- [ ] F2-T3.1 Checkpoint: Component Validation

- Execute pytest suite for `syntax_prettifier` and `table_builder`.

- Verify that 100% of lint results are accounted for in the table mock tests.



- [ ] F2-T4. Integration and CI/CD Readiness

- [ ] F2-T4.1 Construct Rich Report Emitter

- Create `src/adapters/ui/rich_emitter.py`.

- Implement `RichReportEmitter` class.

- Compose `SyntaxHighlighter` and `SummaryTableBuilder` into a single `render_report(results)` method using `rich.console.Console`.

- Use `rich.panel.Panel` for individual error cards.

- _Requirements: F2.1, F2.2_


- [ ] F2-T4.2 Implement CI/CD Compatibility & Environment Detection

- Implement environment detection logic in `RichReportEmitter.get_console()`.

- Use `rich.console.Console(force_terminal=False)` to detect pipes.

- Ensure return codes are propagated correctly from the application layer to the OS.

- Verification: Run in a piped environment (e.g., `linter | cat`) to ensure no crashes and clean output.

- _Requirements: F2.4_



- [ ] F2-T5. Checkpoint 2: Final Verification

- [ ] F2-T5.1 Final Property Alignment & QA

- Perform a manual visual check of the report layout (Requirements E9, E12, E15).

- Execute property test 'CI Transparency' by simulating `TERM=dumb` and piped STDOUT.

- Ensure zero linting violations in the new adapter modules.




## Notes




- Tasks marked with (*) are optional aesthetic enhancements like custom themes.

- Traceability is maintained via requirement_refs to ensure every UI element map back to user needs.

- Ordering constraint: The `syntax_prettifier` must be implemented before the `rich_emitter` to allow integration of code snippets into the final report layout.

- Backward compatibility: The existing CLI output (plain text) must remain the default unless the --rich flag is passed.

