# Calicat Agent Skills

[中文版](./README.md)

A Vercel Skills compatible repository for Calicat-related agent skills.

This repository is designed to be installed with `npx skills add`.

## Install

List available skills:

```bash
npx skills add ./calicat-agent-skills --list
```

Install from a local path while developing:

```bash
npx skills add ./calicat-agent-skills --skill calicat-cli-operator
```

Install from GitHub after the repository is published:

```bash
npx skills add calicatcn/calicat-agent-skills --skill calicat-cli-operator
```

Install globally for Codex:

```bash
npx skills add calicatcn/calicat-agent-skills --skill calicat-cli-operator -g -a codex -y
```

Install globally for Claude Code:

```bash
npx skills add calicatcn/calicat-agent-skills --skill calicat-cli-operator -g -a claude-code -y
```

## Available Skills

### `calicat-cli-operator`

Use the local Calicat CLI to:

- parse Calicat design links correctly
- identify `file_id`, `canvas_id`, and `selected_layer_id`
- fetch canvas/page/layer structure progressively
- retrieve PRD data and interaction design data
- avoid loading oversized design JSON all at once
- check CLI version, update the CLI, or guide installation when needed

## Repository Structure

```text
skills/
  calicat-cli-operator/
    SKILL.md
    references/
      cli-lifecycle.md
```

## Notes

- The repository follows the open agent skills format used by `npx skills`.
- The skill itself is passive context. It is loaded by the agent when the task matches the skill description.
- CLI installation, update, and version-check guidance is intentionally kept in `references/cli-lifecycle.md` so it is loaded only when needed.
