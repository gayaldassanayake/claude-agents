# wso2-trigger-onboarder

A Claude Code agent (plus the `creating-integrator-triggers` skill) that adds the low-code UX for a new
integration **trigger** to the WSO2 Integrator. It drives the changes across all three repos, writes and runs
the language-server tests, and opens three PRs.

## What it does

- Classifies the trigger — event/webhook (e.g. Shopify), database CDC (e.g. MySQL), or message-broker/file
  (e.g. Kafka, RabbitMQ, FTP) — and applies the matching `ballerina-language-server` recipe (Java service
  builder, JSON form model, router registration, artifact/registry/gradle entries, regenerated index)
- Wires the icon in `vscode-extensions` (three icon-mapping edits, a CDC toggle, dark/light + glyph SVGs)
- Duplicates the icon assets into `product-integrator`
- Writes LS tests matching the trigger and runs them until green
- Branches off a user-chosen base branch (local or upstream), pushes to your fork, and opens three PRs
- Pauses at checkpoints: the Ballerina module, the form preview, the icons, and before pushing/PRs

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed
- `gh` CLI installed and authenticated (`gh auth login`)
- Gradle and [Rush](https://rushjs.io/) available for building the language server and VSCode extension
- Local clones (or forks) of `ballerina-language-server`, `vscode-extensions`, and `product-integrator`,
  each with `origin` = your fork and `upstream` = the official repo

## Installation

1. **Copy the agent definition:**
   ```bash
   cp agents/wso2-trigger-onboarder.md ~/.claude/agents/
   ```

2. **Copy the skill suite:**
   ```bash
   cp -r skills/creating-integrator-triggers ~/.claude/skills/
   ```

3. **Restart Claude Code** (or open a new session) — it will auto-discover the agent and skill.

### Choosing the model

The agent ships with `model: inherit`, so it runs on whatever model your current session uses — pick it with
`/model` before invoking the agent (e.g. switch to Opus for the heavier multi-repo runs). To pin a fixed
model instead, edit the `model:` field in `agents/wso2-trigger-onboarder.md` to `sonnet`, `opus`, or `haiku`.

## Usage

Describe the trigger you want to onboard. Examples:

- *"Onboard the ballerinax/trigger.shopify trigger to the WSO2 Integrator"*
- *"Add a MySQL CDC trigger to the low-code editor"*
- *"Wire up a new Kafka-style event trigger across the three repos"*

The agent asks a few questions first (the Ballerina module, a form preview, and icons), then implements the
changes, runs the LS tests, and — after confirming with you — opens the three PRs.

## Files

```
wso2-trigger-onboarder/
├── agents/
│   └── wso2-trigger-onboarder.md            ← agent definition (copy to ~/.claude/agents/)
└── skills/
    └── creating-integrator-triggers/
        ├── SKILL.md                         ← orchestrator (copy to ~/.claude/skills/creating-integrator-triggers/)
        └── references/
            ├── language-server.md           ← ballerina-language-server recipe + LS test pattern
            ├── vscode-extensions.md         ← icon wiring + SVG assets + CDC toggle
            └── product-integrator.md        ← duplicate the icon assets
```
