# Implementation Plan: Rich TUI Visualizations




## Overview




The implementation follows a domain-driven approach, starting with the foundational data models and theme configurations required to support rich terminal output. By defining the 'Diagnostic' and 'Theme' entities first, we ensure that subsequent rendering logic remains decoupled from specific TUI libraries like 'rich'.

The second phase implements the 'Visualization Orchestrator' which serves as the bridge between the core engine results and terminal-specific adapters. Finally, we build out high-fidelity adapters for snippet rendering and diagnostic summaries, concluding with a robust suite of property-based tests to verify visual and statistical accuracy across different terminal environments (TTY vs non-TTY).




## Tasks




- [ ] F3-T1. Domain Model & Theme Definition

- [ ] F3-T1.1 Extend Domain Models and Theme Config

- In src/domain/models.py, create a 'ThemeConfig' dataclass to store ANSI color mappings (e.g., error=red, warning=yellow).

- Extend the 'Diagnostic' model to include 'context_lines' (List[str]) and 'file_uri' (str) for IDE deep-linking support.

- Add 'DiagnosticCategory' Enum to support grouped summaries.

- _Requirements: R1, R2, R4_



- [ ] F3-T2. Use Case Orchestration

- [ ] F3-T2.1 Implement Visualization Orchestrator

- Create src/usecases/visualizer.py with a 'VisualizationOrchestrator' class.

- Implement 'generate_report(diagnostics: List[Diagnostic])' method.

- Logic should handle the logic for calculating aggregate counts by category before passing data to adapters.

- _Requirements: R2, R3_



- [ ] F3-T3. Internal Logic Checkpoint

- [ ] F3-T3.1 Checkpoint: Orchestrator Logic Validation

- Write unit tests for VisualizationOrchestrator to ensure categories are aggregated correctly.

- Verify that Diagnostic objects contain valid line/column metadata.

- _Requirements: R2_



- [ ] F3-T4. TUI Adapter Development

- [ ] F3-T4.1 Develop Syntax-Highlighted Snippet Renderer

- Create src/adapters/tui/snippet_renderer.py.

- Implement 'SnippetRenderer.render()' using syntax highlighting logic.

- Ensure line numbers and pointers (^) are accurately calculated based on Diagnostic coordinates.

- _Requirements: R1_


- [ ] F3-T4.2 Develop Diagnostic Summary Table & Deep-Linking

- Create src/adapters/tui/painter.py.

- Implement 'SummaryTablePainter' to generate a static table of error counts using the 'ThemeConfig'.

- Implement 'DeepLinkFormatter' to generate 'file://' URI schemes compatible with modern terminals (VS Code/PyCharm).

- _Requirements: R2, R4_


- [ ] F3-T4.3 Implement Static Output and TTY Detection

- Add logic to detect TTY status (sys.stdout.isatty()).

- Force 'no_color' and 'plain_text' modes if TTY is false or if NO_COLOR env var is set.

- _Requirements: R3_



- [ ] F3-T5. Final Validation Checkpoint

- [ ] F3-T5.1 Property-Based Verification

- Property Test: Pass 100 random diagnostics and assert that the summary table sum equals 100.

- Property Test: Mock a non-interactive terminal and assert that no ESC [ codes are present in output.

- Property Test: Verify that the '^' character in snippets aligns exactly with the 'column' attribute of the Diagnostic.

- _Requirements: R1, R2, R3_




## Notes




- Tasks marked with (*) are optional optimizations for CI environments.

- Requirements Traceability: Every sub-task is mapped to a specific requirement (R1-R4) to ensure feature parity.

- Ordering Constraint: Domain models (Phase 1) must be finalized before adapters (Phase 3) to ensure type safety across the visualizer.

- Backward Compatibility: The new TUI output will be triggered via a flag; the default stdout behavior remains unchanged for legacy scripts.

