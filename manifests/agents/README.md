# Agent Definitions

This directory contains KAgent Agent resource definitions.

## Creating a New Agent

Use the template in `templates/agent-template.yaml` as a starting point.

## Agent Types

- **Declarative**: Agents defined with system messages and tools
- **Programmatic**: Agents with custom code logic

## Naming Convention

Agent names should:
- Use lowercase letters
- Separate words with hyphens
- Be descriptive of the agent's purpose
- Be unique within the namespace

Examples:
- `kubernetes-operator`
- `github-pr-reviewer`
- `deployment-optimizer`
