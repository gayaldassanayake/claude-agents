# sdk-connector

A Claude Code skill that guides a developer end-to-end through generating a brand-new Ballerina connector from an existing Java SDK, driver, or client library.

## What it does

- Runs a feasibility check before committing to an SDK-based approach (vs. OpenAPI, a simple HTTP wrapper, or a webhook)
- Explains the domain and negotiates connector scope with the developer, capability by capability
- Evaluates and recommends an SDK when more than one candidate exists
- Explores the chosen SDK's capabilities
- Designs the public Ballerina API surface (`spec.md`) before any implementation exists, and identifies where a compiler plugin is warranted
- Scaffolds the connector repository from the [sdk-connector-template](https://github.com/gayaldassanayake/sdk-connector-template)
- Writes the test suite before implementation, then implements against it until green
- Verifies test coverage, writes standalone examples, and completes documentation
- Optionally verifies GraalVM native-image compatibility

Each phase is a hard gate: the skill gets explicit developer sign-off before moving to the next.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed
- `gh` CLI installed and authenticated (`gh auth login`), if scaffolding a new repository
- Gradle and the Ballerina distribution available for the implementation/test phases

## Installation

1. **Copy the skill:**
   ```bash
   cp -r skills/sdk-connector ~/.claude/skills/
   ```

2. **Restart Claude Code** (or open a new session) — it will auto-discover the skill.

## Usage

Invoke the skill by asking Claude Code to build a connector from an SDK. Examples:

- *"Build a Ballerina connector for the Foo Java SDK"*
- *"I want to wrap this vendor's Java client as a Ballerina connector"*

Do not use this skill if the service already has an OpenAPI specification, or can be implemented as a simple HTTP wrapper, webhook, or a patch to an existing connector.

## Files

```
sdk-connector/
└── skills/
    └── sdk-connector/
        ├── SKILL.md                       ← skill definition (copy to ~/.claude/skills/sdk-connector/)
        └── references/
            └── artifact-generation.md      ← conventions for the HTML/markdown artifacts each phase produces
```
