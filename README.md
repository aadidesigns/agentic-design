# Agentic Design Workflow

A token-optimised, gate-driven design automation system built on Claude Code. You provide a requirements file — optionally paired with a Figma URL or inspiration references — and the system walks you through planning, wireframing, implementation, and QA with a human approval gate at every meaningful decision point.

## Install

```
/plugin marketplace add aadidesigns/agentic-design
/plugin install agentic-design-workflow@aadidesigns
```

## How to start a session

Open Claude Code and invoke the agent:

```
@design-planner I have a new requirement
```

Paste or reference your `requirements.md` file. The agent takes it from there.

## What the workflow does

The system runs in four layers. First, the planner reads your requirements and produces a structure brief (layout and component inventory only) for your approval. Once approved, it expands into a full design spec with typography, spacing, colour, and per-task effort estimates. You pick which tasks to build. For each task, you choose whether to build in Figma first (via MCP) or go straight to React/TypeScript. After implementation, an optional QA pass checks visual consistency, accessibility, edge cases, and requirements coverage — surfacing issues with effort costs so you decide what's worth fixing.

## Agents and skills

| File | Role |
|---|---|
| `agents/design-planner.md` | Session orchestrator — reads inputs, produces briefs and specs, delegates to skills, manages gates |
| `skills/design-engineer/SKILL.md` | Figma implementer — builds frames via MCP from the design spec |
| `skills/code-engineer/SKILL.md` | React/TypeScript implementer — builds components from spec or approved Figma frames |
| `skills/design-qa/SKILL.md` | Visual and functional auditor — checks builds against spec and requirements |
| `skills/ux-writer/SKILL.md` | Copy and label reviewer — checks UI strings for length, tone, and truncation risk |

## Prerequisites

- **Claude Code** with agent and skill support
- **Figma MCP** (optional) — required only if you want the Figma-first implementation path
- **React/TypeScript project** — required for the code-engineer skill to build and serve components

### Recommended permissions

Add these to your `.claude/settings.local.json` if you plan to use Figma MCP:

```json
{
  "permissions": {
    "allow": [
      "mcp__figma__get_design_context",
      "mcp__figma__get_metadata",
      "mcp__figma__get_variable_defs",
      "mcp__figma__get_screenshot",
      "mcp__figma__create_frame",
      "mcp__figma__update_frame",
      "Bash(npm run dev)",
      "Bash(npm run build)",
      "Bash(npx tsc --noEmit)",
      "Bash(find:*)",
      "Bash(open:*)"
    ]
  }
}
```

## Working directories

On first run, the `design-planner` agent scaffolds these directories in your project:

```
inputs/          — your requirements and inspiration files
memory/          — session state tracking
outputs/briefs/  — structure briefs and design specs
outputs/builds/  — built components
outputs/figma-logs/ — Figma token maps and gap reports
outputs/qa-logs/ — QA and copy review logs
```

## License

MIT
