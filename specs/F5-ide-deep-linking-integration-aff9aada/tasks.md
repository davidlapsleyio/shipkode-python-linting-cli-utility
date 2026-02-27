# Implementation Plan: IDE Deep-Linking Integration




## Overview




The implementation strategy follows a 'Domain-first' then 'Adapter-extension' approach. We first define the core data models for URI schemes and coordinate mapping, ensuring that the link generation logic is decoupled from the terminal rendering logic. This prevents 'leaky abstractions' where terminal-specific escape codes bleed into the core logic.

The first phase focuses on the Link Generator adapter, creating a robust engine to translate file paths and coordinates into IDE-specific URIs (e.g., vscode://, pycharm://). Next, we refactor the existing Terminal Reporter to support 'pluggable' path decorators. This allows us to inject the OSC 8 hyperlink logic without refactoring the entire reporting loop. Finally, we implement property-based tests to ensure the generated escape sequences are valid and that coordinates are accurately aligned between the UI text and the hidden URI metadata.




## Tasks




- [ ] F5-T1. Core Link Generation Logic

- [ ] F5-T1.1 Implement IDE Link Generator adapter

- Create `src/adapters/link_generator.py`.

- Implement `LinkGenerator` class with support for `vscode://`, `pycharm://`, and `file://` protocols.

- Add method `generate_link(path: str, line: int, col: int) -> str` that reads from an `IDE_PROTOCOL` environment variable.

- Ensure proper URI encoding for file paths using `urllib.parse.quote`.

- _Requirements: F5-3_


- [ ] F5-T1.2 Create OSC 8 Escape Sequence Utilities

- Add a `HyperlinkFormatter` utility in `src/adapters/terminal_utils.py`.

- Implement OSC 8 sequence wrapper: `\033]8;;{uri}\033\\{text}\033]8;;\033\\`.

- Include a toggle for `TERM_SUPPORT_OSC8` to allow for graceful degradation.

- _Requirements: F5-1_



- [ ] F5-T2. Adapter Integration and UI Enhancement

- [ ] F5-T2.1 Enhance Terminal Reporter for Clickable Links

- Open `src/adapters/terminal_reporter.py`.

- Identify the violation row rendering loop.

- Refactor path rendering to use the `LinkGenerator` to wrap the filename and coordinate text in OSC 8 sequences.

- Ensure the 'Summary Table' headers and rows are updated to pass file/line/col objects instead of pre-formatted strings.

- _Requirements: F5-2_


- [ ] F5-T2.2 Update Summary Table Metadata Rendering Logic

- Update the summary table logic to include high-precision coordinates for Security, Style, and Syntax errors.

- Ensure visual hierarchy is maintained as per Requirement 2.

- _Requirements: F5-2_



- [ ] F5-T3,sub_tasks:[{description:. Checkpoint: Verify OSC 8 Output in Supported Terminal

- Validate the integration and property adherence.


- [ ] F5-T3.2. Property-Based Verification Tests

- Create `tests/test_link_integrity.py`.

- Property Test: Verify that for a given violation, the line/col numbers in the URI match the line/col numbers in the visible text (Coordinate Precision).

- Property Test: Verify that changing `IDE_PROTOCOL` env var changes the URI scheme prefix in output (Protocol Adherence).

- Property Test: Verify that when OSC8 is disabled, no escape sequences starting with `\033]8` are present (Graceful Degradation).


- [ ] F5-T3.3. Manual E2E Validation

- Run the linter on its own codebase.

- Manually click links in a supported terminal (e.g., iTerm2, VSCode Integrated Terminal, Alacritty) to confirm IDE focus switch.

- _Requirements: F5-1_


- [ ] F5-T3.4. Final Checkpoint

- Verify zero regression in standard terminal output for non-supported environments.

- Confirm all tests pass.



## Notes




- Tasks marked with (*) are optional based on specific terminal emulator capabilities.

- Traceability is maintained by mapping every sub-task to its parent requirement (F5-R1/R2/R3).

- The 'Split-First' strategy is applied to the Terminal Reporter to ensure the new logic doesn't bloat the existing reporting engine.

- Backward compatibility: The 'Graceful Degradation' property ensures that older terminals see standard text paths if the OSC 8 toggle is disabled or unsupported.

