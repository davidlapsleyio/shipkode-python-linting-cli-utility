# Implementation Plan: CI/CD Packaging & Actions




## Overview




The implementation strategy follows a 'Distribution-First' approach, where the core Python package is stabilized before being wrapped into higher-level delivery mechanisms. We first establish the pyproject.toml configuration to ensure the tool is installable and executable. This forms the foundation for the Slim Docker image, which leverages the pip-installed package to minimize layer size. Finally, the GitHub Action is built as a wrapper around the Docker image to provide a turnkey CI/CD solution.

The rationale for this order is dependency management: the Docker image cannot be verified without a working package, and the GitHub Action cannot function without a stable container image. This incremental build-up ensures that each layer of the delivery stack is validated against its specific correctness properties (Executability, Footprint, and Gate Enforcement) before moving to the next level of abstraction.




## Tasks




- [ ] F6-T1. Python Distribution Packaging

- [ ] F6-T1.1 Configure pyproject.toml and Entry Points

- Create/Update `pyproject.toml` using building backend (e.g., hatchling or setuptools).

- Define `[project.scripts]` mapping `pyvisualguard = "pyvisualguard.cli:main"`.

- Explicitly list all production dependencies in `dependencies` array.

- Configure clean build paths to exclude tests/ and docs/ from the sdist/wheel.

- _Requirements: 1_


- [ ] F6-T1.2 Verify Package Executability Property

- Create a scripted verification test.

- Create a fresh virtual environment.

- Run `pip install .` followed by `pyvisualguard --version`.

- Verify no development-only dependencies (like pytest) are installed.

- _Requirements: 1_



- [ ] F6-T2. Phase 1 Checkpoint

- [ ] F6-T2.1 Build and Inspect Distribution Artifacts

- Run `python -m build` to generate .tar.gz and .whl.

- Use `twine check` to verify metadata.

- Perform a dry-run install from the generated wheel.

- _Requirements: 1_



- [ ] F6-T3. CI-Ready Containerization

- [ ] F6-T3.1 Implement Multi-Stage Slim Dockerfile

- Create a multi-stage `Dockerfile`.

- Stage 1 (Builder): Install build-essential and compile necessary wheels.

- Stage 2 (Final): Use `python:3.11-slim` or `alpine`.

- Copy only the compiled wheels/installed package from Stage 1 to Stage 2.

- Set `ENTRYPOINT ["pyvisualguard"]`.

- _Requirements: 2_


- [ ] F6-T3.2 Verify Minimal Image Footprint Property

- Build the image: `docker build -t pyvisualguard:latest .`.

- Inspect image size using `docker images` to ensure it is < 150MB.

- Run `docker run --rm pyvisualguard:latest --help` to verify execution.

- Scan image for presence of 'gcc' or 'git' to ensure no leak of build tools.

- _Requirements: 2_



- [ ] F6-T4. GitHub Action Integration

- [ ] F6-T4.1 Define GitHub Action Metadata and Interface

- Create `action.yml` in the repository root.

- Define inputs: `path`, `fail-on-critical`, `output-format`.

- Configure the action to run via `docker` using the image built in Phase 3.

- Map the GitHub workspace to the container volume.

- _Requirements: 3_


- [ ] F6-T4.2 Verify Quality Gate Enforcement Property

- Create a test repository/workflow that invokes the local action.

- Scenario A: Scan a codebase with a known critical vulnerability and `fail-on-critical: true`.

- Scenario B: Scan the same codebase with `fail-on-critical: false`.

- Verify exit code is 1 for Scenario A and 0 for Scenario B.

- _Requirements: 3_



- [ ] F6-T5. Final Checkpoint

- [ ] F6-T5.1 End-to-End Orchestration Test

- Execute the full CI pipeline including package build, docker push, and action test.

- Confirm all artifacts (PyPI wheel, Docker image, GH Action) are synchronized in versioning.

- _Requirements: 1, 2, 3_




## Notes




- Tasks marked with (*) are optional based on specific runner environment constraints.

- Traceability is maintained by mapping every sub-task to a specific Requirement ID (e.g., E4) and Property ID.

- The 'split-first' strategy is applied by isolating the packaging logic into pyproject.toml before containerization.

- Backward compatibility is guaranteed by maintaining the CLI entry point signature across all distribution formats.

