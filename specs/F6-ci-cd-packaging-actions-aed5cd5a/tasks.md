# Implementation Plan: CI/CD Packaging & Actions




## Overview




The implementation strategy follows an 'Infrastructure-as-Code' and 'Adapter-First' approach. We begin by defining the core packaging and structured output capabilities, as these form the foundation for all distribution channels. By establishing the pyproject.toml and JSON formatters first, we ensure that the subsequent Docker and GitHub Action layers are thin wrappers around a stable, verifiable core.

The plan moves from the internal Python ecosystem (PyPI) to the containerization layer (Docker), and finally to the orchestration layer (GitHub Actions). This sequence allows us to validate correctness properties—such as exit codes and JSON schemas—at the lowest level before they are inherited by the higher-level CI components. Checkpoints are strategically placed after each distribution tier to verify image sizes and schema compliance.




## Tasks




- [ ] F6-T1. Core Packaging & Formatter Foundation

- [ ] F6-T1.1 Standardize Python Package Manifest

- Create `pyproject.toml` using setuptools or hatch backend.

- Define `[project.scripts]` entry point mapping `pyvisualguard` to `pyvisualguard.cli:main`.

- Specify dependencies: `click`, `pydantic`, `pyyaml`.

- Verification: Run `pip install .` and verify the `pyvisualguard` command is available in the PATH.

- _Requirements: 1_


- [ ] F6-T1.2 Implement JSON Structured Log Formatter

- Enhance `src/pyvisualguard/adapters/formatters.py` with a `JSONFormatter` class.

- Implement a `serialize()` method that converts `ScanResult` domain objects into JSON strings.

- Ensure fields like `vulnerability_id`, `severity`, and `file_path` are included.

- Verification: Run `pyvisualguard --format json` and pipe to `jq .` to validate structure.

- _Requirements: 4_


- [ ] F6-T1.3 Develop Reliable Gatekeeping Exit Logic

- Update the CLI entry point in `src/pyvisualguard/interfaces/cli.py`.

- Implement logic to catch domain exceptions and return exit code 1 if `severity == 'CRITICAL'`.

- Ensure exit code 0 for clean runs or low-severity findings.

- Verification: Execute against a known vulnerable directory and assert `$?` (exit status) is 1.

- _Requirements: 3_



- [ ] F6-T2. Checkpoint: Core Validation

- [ ] F6-T2.1 Verification: Core Distribution Checkpoint

- Execute property test `Reliable Gatekeeping Exit Codes (CP1)`: Verify exit 1 on critical findings.

- Execute property test `Structured Output Verifiability (CP2)`: Validate JSON output against the Pydantic schema.

- Run `tox` or `pytest` to ensure distribution metadata is valid.



- [ ] F6-T3. Containerization & Optimization

- [ ] F6-T3.1 Develop Slim CI Docker Image

- Define `Dockerfile` using `python:3.11-slim` as the base image.

- Implement multi-stage build: build wheel in stage 1, install in stage 2 to minimize footprint.

- Remove build-time dependencies (`gcc`, etc.) and clear `pip` cache.

- Set `ENTRYPOINT ["pyvisualguard"]`.

- _Requirements: 2_


- [ ] F6-T3.2 Verify Docker Image Footprint

- Execute property test `Image Size Constraint (CP3)`.

- Run `docker build -t pyvisualguard:latest .`.

- Run `docker images` and verify size is < 150MB.



- [ ] F6-T4. GitHub Actions Integration

- [ ] F6-T4.1 Create GitHub Action Wrapper

- Create `action.yml` in the repository root.

- Map inputs (path, format, threshold) to CLI arguments.

- Use `docker` execution type referencing the Slim CI image.

- Map GitHub workspace to the container filesystem.

- _Requirements: 3_


- [ ] F6-T4.2 Verify Action Integration Gatekeeping

- Create `.github/workflows/test-action.yml`.

- Configure it to run on a local 'fixtures' directory containing known vulnerabilities.

- Assert that the workflow fails when critical issues are found and passes otherwise.

- _Requirements: 3_




## Notes




- Tasks marked with (*) are optional based on specific CI provider needs but recommended for completeness.

- Traceability: Sub-tasks explicitly reference requirements (e.g., F6-R1) and properties (e.g., CP1) to ensure every design goal is accounted for.

- Ordering Constraint: The JSON formatter must be implemented before the GitHub Action to ensure the action can provide structured feedback to the GH UI.

- Compatibility: The pyproject.toml configuration ensures backward compatibility with older pip versions via build-system requirements.

