# ballerina-bug-fixer

A Claude Code agent that autonomously diagnoses and fixes bugs in Ballerina connector modules, verifies that fixes build cleanly, and opens a PR to the upstream repository.

## What it does

- Handles CVE / dependency vulnerability fixes (bumps versions in `gradle.properties`)
- Fixes compiler type errors, runtime panics, build failures, and test failures
- Reads the project's README for build instructions before making any changes
- Applies minimal, targeted fixes and adds/updates tests where applicable
- Creates a branch, commits, pushes to your fork, and opens a PR

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed
- `gh` CLI installed and authenticated (`gh auth login`)
- Gradle available in your PATH (for building Ballerina connector projects)

## Installation

1. **Copy the agent definition:**
   ```bash
   cp agents/ballerina-bug-fixer.md ~/.claude/agents/
   ```

2. **Copy the Ballerina skill suite:**
   ```bash
   cp -r skills/ballerina ~/.claude/skills/
   ```

3. **Restart Claude Code** (or open a new session) — it will auto-discover the agent and skills.

## Usage

Invoke the agent by describing a Ballerina bug in Claude Code. Examples:

- *"Fix the CVE GHSA-72hv-8253-57qq in this repo"*
- *"There's a type error in `ballerina/caller.bal` on line 42"*
- *"The build is failing with a TOML misconfiguration"*
- *"This GitHub issue describes a runtime panic: \<url\>"*

The agent will ask a few pre-flight questions (whether to run tests, credential setup, target branch) before touching any code.

## Files

```
ballerina-bug-fixer/
├── agents/
│   └── ballerina-bug-fixer.md       ← agent definition (copy to ~/.claude/agents/)
└── skills/
    └── ballerina/
        ├── SKILL.md                 ← Ballerina language skill (copy to ~/.claude/skills/ballerina/)
        └── references/
            ├── debugging.md         ← compiler/runtime error reference
            └── integration-patterns.md  ← DB, Kafka, GraphQL, gRPC patterns
```
