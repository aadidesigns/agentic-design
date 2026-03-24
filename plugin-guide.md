# Requirements: Package agentic-design-workflow as a Claude Code Plugin

## Objective

Take the existing agentic design workflow (agents + skills) and restructure it into a valid Claude Code plugin that can be distributed via a plugin marketplace. The plugin should be installable with:

```
/plugin marketplace add aadityabasnyat/agentic-design-workflow
/plugin install agentic-design-workflow@aadityabasnyat
```

---

## Current structure (what exists)

The workflow currently uses this layout:

```
.claude/
├── agents/
│   └── design-planner.md
└── skills/
    ├── design-engineer/
    │   └── SKILL.md
    ├── code-engineer/
    │   └── SKILL.md
    ├── design-qa/
    │   └── SKILL.md
    └── ux-writer/
        └── SKILL.md
```

Plus working directories:

```
inputs/
├── requirements.md
└── inspiration/

memory/
└── session.md

outputs/
├── briefs/
├── figma-logs/
├── builds/
└── qa-logs/
```

---

## Target structure (what to produce)

Restructure into a Claude Code plugin following the official plugin spec:

```
agentic-design-workflow/
│
├── .claude-plugin/
│   └── plugin.json
│
├── agents/
│   └── design-planner.md
│
├── skills/
│   ├── design-engineer/
│   │   └── SKILL.md
│   ├── code-engineer/
│   │   └── SKILL.md
│   ├── design-qa/
│   │   └── SKILL.md
│   └── ux-writer/
│       └── SKILL.md
│
└── README.md
```

---

## plugin.json spec

Create `.claude-plugin/plugin.json` with these fields:

```json
{
  "name": "agentic-design-workflow",
  "description": "A token-optimised, gate-driven design automation system. Four specialised agents orchestrate planning, Figma implementation, React builds, and QA — with human approval at every step.",
  "version": "1.0.0",
  "author": {
    "name": "Aaditya Basnyat"
  },
  "homepage": "https://aaditya.design",
  "repository": "https://github.com/aadityabasnyat/agentic-design-workflow",
  "license": "MIT",
  "keywords": [
    "design",
    "figma",
    "react",
    "ux",
    "design-system",
    "automation",
    "qa"
  ],
  "agents": [
    "./agents/design-planner.md"
  ]
}
```

**Notes on plugin.json:**
- `agents` array points to the entry-point agent. Skills are invoked by the agent — they don't need to be listed separately since Claude Code auto-discovers skills inside the `skills/` directory.
- Do NOT include `commands`, `hooks`, `mcpServers`, or `lspServers` — this plugin uses none of those.

---

## Agent file requirements

### `agents/design-planner.md`

Keep the existing frontmatter and prompt content from the workflow spec with these adjustments:

1. **Remove `memory: project`** from frontmatter — plugins don't own project memory. The agent writes to `memory/session.md` in the user's working directory instead.
2. **Remove `model: claude-sonnet-4-5`** from frontmatter — let the user's default model apply, or document it as a recommendation in the README.
3. The agent's first action on invocation should be to **scaffold the working directories** (`inputs/`, `memory/`, `outputs/` and their subdirectories) in the user's current project if they don't already exist.
4. The agent should create `inputs/requirements.md` and `memory/session.md` with the starter content from the workflow spec if they don't already exist.

Frontmatter should be:

```yaml
---
name: design-planner
description: "Invoke this agent to start any new design session. Use @design-planner when you have a requirements file and want to plan, brief, spec, or implement a design. This agent orchestrates all other skills — do not invoke design-engineer, code-engineer, design-qa, or ux-writer directly unless explicitly told to."
---
```

---

## Skill file requirements

All four skills (`design-engineer`, `code-engineer`, `design-qa`, `ux-writer`) should be carried over exactly as defined in the workflow spec. No changes to frontmatter or prompt content needed — skills are self-contained.

Verify each skill file has valid YAML frontmatter with at minimum:
- `name`
- `description`

Remove `$ARGUMENTS` from the end of each skill file if present — this is a placeholder convention from the spec, not valid plugin syntax.

---

## README.md

Write a README that covers:

1. **What this is** — one paragraph, plain language
2. **Install** — the marketplace add + plugin install commands
3. **How to start a session** — invoke `@design-planner` with a requirements file
4. **What the workflow does** — the four layers (input → structure brief → design spec → implementation → QA) described in 4–6 sentences, not a full spec dump
5. **Agents and skills** — a table listing all five files with one-line descriptions
6. **Prerequisites** — Figma MCP (optional, for Figma-first path), a React/TS project (for code-engineer)
7. **Working directories** — explain that the plugin scaffolds `inputs/`, `memory/`, `outputs/` in the user's project on first run
8. **License** — MIT

Do NOT reproduce the full workflow spec in the README. Keep it concise. Link to the spec or a docs page for the full breakdown.

---

## Marketplace file (separate repo)

If the plugin repo also serves as its own marketplace, include a marketplace.json:

```
agentic-design-workflow/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
```

```json
{
  "name": "aadityabasnyat",
  "owner": {
    "name": "Aaditya Basnyat"
  },
  "metadata": {
    "description": "Design automation plugins by Aaditya Basnyat"
  },
  "plugins": [
    {
      "name": "agentic-design-workflow",
      "source": ".",
      "description": "A token-optimised, gate-driven design automation system with four specialised agents, human approval gates, and Figma MCP integration."
    }
  ]
}
```

**Note:** `source: "."` works because the plugin lives at the root of the marketplace repo. This is valid for a single-plugin marketplace.

---

## What NOT to include in the plugin

- `settings.local.json` — this is user-side config, not plugin content. Document the recommended permissions in the README instead so users can add them to their own settings.
- The `inputs/`, `memory/`, `outputs/` directories — these are scaffolded at runtime by the agent in the user's project, not shipped with the plugin.
- Any node_modules, build artifacts, or lock files.

---

## Validation checklist

Before publishing, the agent should verify:

- [ ] `.claude-plugin/plugin.json` is valid JSON with all required fields
- [ ] `.claude-plugin/marketplace.json` is valid JSON (if included)
- [ ] `agents/design-planner.md` has valid YAML frontmatter
- [ ] All four skill files have valid YAML frontmatter
- [ ] No `$ARGUMENTS` placeholders remain in any file
- [ ] No references to `.claude/` paths (old convention) — all paths are relative to plugin root
- [ ] `README.md` exists and is not a copy of the full workflow spec
- [ ] Run `claude plugin validate .` or `/plugin validate .` and confirm no errors