# Cognition-Compiler

`Cognition-Compiler` is a repository of reusable skills for GitHub Copilot style agent workflows.

The current repository focuses on three capabilities:

- maintaining reusable instructions from user corrections
- scaffolding new skills from a shared template
- understanding a codebase and generating repository-specific instructions

## Repository Layout

```text
skills/
  instruction-maintainer/
    SKILL.md
  skill-builder/
    SKILL.md
    templates/
      skill-template.md
  understand-codebase/
    SKILL.md
    reference/
      1-determine-techstack.md
      2-categorize-files.md
      3-identify-architecture.md
      4-domain-deep-dive.md
      5-styleguide-generation.md
      6-build-instructions.md
      api/
        *.yml
```

## Included Skills

### `instruction-maintainer`

Detects stable, reusable corrections in user feedback and helps persist them into instruction files at the smallest correct scope.

Use this when you want to turn repeated user corrections into durable repository or scoped Copilot instructions.

### `skill-builder`

Creates new skills using a consistent template-first workflow.

Use this when you want to add a new skill that matches the structure, writing style, and modular conventions already used in this repository.

### `understand-codebase`

Analyzes a repository through a prompt chain and produces slim root instructions plus more specific scoped instruction files.

This skill includes reference prompts for:

- determining the tech stack
- categorizing files
- identifying architecture
- performing domain deep dives
- generating style guides
- building final repository instructions

It also includes API specification examples under `reference/api/` for external API shape guidance.

## Design Principles

This repository follows a few consistent patterns across skills:

- template-first authoring over ad hoc structure
- minimal, composable sections instead of large monolithic instructions
- scoped behavior rules instead of bloated global guidance
- repository evidence before generic assumptions

## Adding A New Skill

1. Start from `skills/skill-builder/templates/skill-template.md`.
2. Reuse the section structure and naming conventions from existing skills.
3. Keep the new skill narrowly scoped to one reusable capability.
4. Add supporting templates or reference material only when the skill actually needs them.

## Intended Use

This repository is meant to be used as a source of agent skills and prompt-chain building blocks. It is most useful when you want to standardize how an AI coding assistant:

- learns from corrections
- generates new reusable skills
- studies a codebase before writing project-specific instructions

## Status

The repository is now versioned in Git and published on GitHub at:

- <https://github.com/nrh387/Cognition-Compiler>