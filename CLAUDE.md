# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RepoSwarm is an AI-powered multi-repository architecture discovery platform that analyzes GitHub repositories using Claude and generates `.arch.md` architecture documentation files. It uses Temporal for workflow orchestration and can store results in DynamoDB or a file-based system.

## Development Commands

All commands use `mise` (a tool version manager). Prefer mise tasks over custom bash commands.

### Setup
```bash
mise get-started          # Interactive setup wizard (creates .env.local)
mise dev-dependencies     # Install Python dependencies (uv sync)
```

### Running Investigations
```bash
mise investigate-all                    # Analyze all repos from repos.json (starts server, worker, client)
mise investigate-one                    # Analyze single repo (defaults to "is-odd")
mise investigate-one is-even            # Analyze specific repo from repos.json
mise investigate-one <github-url>       # Analyze any GitHub URL
mise investigate-one <repo> force       # Force investigation ignoring cache
```

### Testing
```bash
mise test-all             # Run all tests (unit + integration)
mise test-units           # Run unit tests only (quick feedback)
mise test-integration     # Run integration tests (excludes slow tests)
```

### Development Server
```bash
mise dev-server           # Start Temporal server
mise dev-worker           # Start Temporal worker
mise dev-client           # Trigger workflow client
mise kill                 # Stop all Temporal processes
```

### Monitoring & Debugging
```bash
mise verify-config                              # Validate configuration and test repo access
mise monitor-workflow <workflow_id>             # Check workflow status
mise monitor-temporal                           # Check Temporal server status
```

### Cleanup
```bash
mise cleanup-temp         # Clean temp folder and investigated repositories
```

## Architecture

### Core Components

1. **Investigator** (`src/investigator/`): The analysis engine that clones repos and uses Claude to analyze them
   - `investigator.py`: Main `ClaudeInvestigator` class orchestrating the analysis pipeline
   - `core/claude_analyzer.py`: Handles Claude API interactions
   - `core/repository_type_detector.py`: Auto-detects repo type (backend, frontend, mobile, etc.)
   - `core/file_manager.py`: Manages file operations and prompt reading

2. **Temporal Workflows** (`src/workflows/`): Orchestration layer
   - `investigate_repos_workflow.py`: Main workflow that processes multiple repositories in parallel chunks
   - `investigate_single_repo_workflow.py`: Child workflow for individual repository analysis

3. **Activities** (`src/activities/`): Temporal activity implementations
   - `investigate_activities.py`: Activities for repo analysis, config reading, Claude API calls
   - `investigation_cache.py`: Cache management for avoiding redundant analysis

4. **Prompts** (`prompts/`): AI analysis prompts organized by repository type
   - Each type (backend, frontend, mobile, libraries, infra-as-code, generic) has its own `prompts.json` defining analysis steps
   - `shared/`: Cross-cutting prompts (authentication, authorization, security)
   - `prompt_selector.json`: Rules for automatic type detection

### Workflow Execution Flow

1. Workflow reads `prompts/repos.json` for repositories to analyze
2. For each repo, a child workflow is spawned
3. Repository is cloned, type is detected
4. Prompts are loaded based on type and executed sequentially
5. Results are cached in DynamoDB/file system
6. Generated `.arch.md` is committed to the configured results repository

### Key Configuration

- `.env.local`: Local environment variables (ANTHROPIC_API_KEY, GITHUB_TOKEN, etc.)
- `prompts/repos.json`: List of repositories to analyze
- `mise.toml`: All available tasks and their documentation

## Code Style Guidelines

- Be pragmatic and to the point. Avoid cheerleading or overly optimistic phrasing.
- Use neutral phrasing: "The failure seems to be..." rather than "Great news: it almost works!"
- Avoid exclamation points or overly upbeat phrasing.
- Prefer clarity and simplicity. Confirm understanding before coding.
- Provide concrete, actionable diagnostic steps before suggesting code changes.
- Ask clarifying questions if context is missing.
