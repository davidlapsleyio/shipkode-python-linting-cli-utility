# AGENTS.md

This document provides essential context, architectural constraints, and operational guidelines for AI coding assistants working on the `kando` repository.

## Project Overview
Kando is a production-ready autonomous agent framework. It enables the creation of agents that execute tasks within ephemeral Docker containers, providing environment parity across different programming languages. The system bridges a modern CLI experience with a robust Django 5.x backend.

## Tech Stack
- **Primary Language:** Python 3.12+ (Core logic and Django)
- **Secondary Languages:** Go, TypeScript, JavaScript (Supported guest environments)
- **Framework:** Django 5.x (Internal orchestration engine)
- **CLI:** Typer & Rich (User interface)
- **Infrastructure:** Docker (Container-first execution)
- **AI Integration:** AWS Bedrock, Anthropic (via `apps/ai`)

## Architecture
Kando follows a **12-factor application** methodology.
1.  **Entry Point:** The `kando` CLI script and `kando_cli.py` act as the primary interface.
2.  **Orchestration Engine:** The CLI delegates heavy lifting to Django via management commands (`apps/core/management/commands/kando.py`).
3.  **Modular Core:**
    *   `apps/core`: Agent logic, tool registry, and container management.
    *   `apps/ai`: Model routing, adapters for LLM providers, and language discovery logic.
    *   `apps/api`: REST API scaffolding (Future).

## Key Directories
- `apps/`: Main application source code.
- `.kiro/`: System-specific steering files, Git hooks, and automation specifications. 
- `.kiro/specs/`: Requirements and design docs for specific features (Reference these for historical context).
- `docs/`: Development and architecture guides (e.g., `docs/DEVELOPMENT.md`).
- `examples/`: Reference projects for Python, Go, and TypeScript.
- `kando`: The main executable script.

## Conventions
- **Development Workflow:** Use the `Makefile` for common tasks (build, test, lint).
- **Steering Files:** This project uses "Steering Documentation." Before implementing a feature, check the relevant sub-directory in `.kiro/specs/`. Git hooks in `.kiro/hooks/` automate updates to these documents.
- **Project Discovery:** Language detection logic resides in `apps/ai/detect.py`. When adding support for new languages, follow the pattern of checking for manifests (`package.json`, `go.mod`).
- **Django Implementation:** Business logic should reside in Django apps, not in the CLI entry scripts. Use management commands as the bridge.

## Testing
- **Primary Test Runner:** Jest.
- **CI/CD:** GitHub Actions workflows are defined in `.github/workflows/`.
- **Integration Tests:** Focus on container lifecycle and tool execution parity.
- **Commands:** Run `make test` to execute the suite.

## Constraints
- **Container-First Execution:** You must not write code that assumes dependencies are installed on the host machine. All tool execution should be proxied through Docker containers (see `Dockerfile` and `apps/core`).
- **Django-Powered CLI:** Do not write pure Python scripts for complex tasks. Register a new Django management command and call it from the Typer CLI.
- **Zero Host Dependencies:** The host machine should only require Python/Docker to run the framework; all guest language runtimes (Node, Go compiler) must live within containers.
- **Documentation Consistency:** Changes to fundamental architecture must be reflected in the `.kiro/specs/` directory.

## Key Files
- `kando_cli.py`: The Typer-based entry point for the user.
- `apps/core/management/commands/kando.py`: The bridge where CLI calls enter the Django environment.
- `apps/ai/adapters/`: Contains the logic for different LLM provider interfaces.
- `.env.example`: Template for required environment variables (API keys, etc.).
- `pyproject.toml`: Python dependency and packaging configuration.
- `.kiro/hooks/spell-check.kiro.hook`: Example of the automated verification pipeline.