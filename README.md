# claude-agents

A collection of custom agents and skills for [Claude Code](https://claude.ai/claude-code).

## Agents

| Agent | Description |
|-------|-------------|
| [ballerina-bug-fixer](./ballerina-bug-fixer/) | Diagnoses and fixes bugs in Ballerina connector modules, verifies builds, and opens PRs |
| [wso2-trigger-onboarder](./wso2-trigger-onboarder/) | Adds the low-code UX for a new WSO2 Integrator trigger across three repos, writes and runs LS tests, and opens three PRs |
| [sdk-connector](./sdk-connector/) | Guides a developer end-to-end through generating a brand-new Ballerina connector from an existing Java SDK, driver, or client library |

## How to install an agent

Each agent directory contains its own `README.md` with specific installation instructions. The general pattern is:

1. Copy the agent definition file to `~/.claude/agents/`
2. Copy any accompanying skill files to `~/.claude/skills/`
3. Restart Claude Code (or open a new session)

Claude Code will automatically discover agents in `~/.claude/agents/` and skills in `~/.claude/skills/`.
