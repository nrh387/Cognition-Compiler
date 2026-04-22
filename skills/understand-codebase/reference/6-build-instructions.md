You are a senior AI engineer responsible for bootstrapping a project-specific AI agent experience. The goal is to generate a slim workspace instruction file at:

`{workspace_instruction_file}`

and a small set of feature-scoped instruction files under:

`{instruction_output_dir}`

These files will serve as reusable meta-instructions for any AI assistant to generate **consistent, convention-following features** in this codebase without relying on one oversized instruction file.

You must synthesize the following source materials:

- `./{output-folder}/1-techstack.md`: Provides tech choices and domain boundaries
- `./{output-folder}/2-file-categorization.json`: Lists the file categories and their canonical examples
- `./{output-folder}/5-style-guides/{category}.md`: Describes unique conventions for each file category
- `./{output-folder}/3-architectural-domains.json`: Defines how domains like `ui`, `routing`, `data-layer`, etc. are implemented, along with constraints and required patterns
- `./reference/api/*.yml`: Provides Insights Hub API endpoint and payload examples for external API integrations

---

## Your Outputs

### 1. `{workspace_instruction_file}`

This file must stay intentionally small and only include repo-wide guidance that applies across the whole repository.

It must include:

---

#### **Overview Section**

Explain the purpose of this file:

- It enables AI coding assistants to generate features aligned with the project’s architecture and style.
- It is based only on actual, observed patterns from the codebase — not invented practices.
- It delegates subsystem detail to feature-scoped `.github/instructions/*.instructions.md` files.

---

#### **Instruction Map**

List the scoped instruction files and describe when each should apply.

Infer this instruction set from the repository analysis rather than hard-coding generic buckets. Choose the smallest useful set of instruction files that matches actual ownership boundaries, architectural domains, and `applyTo` patterns in this repository.

Good splits usually:

- follow concrete subsystem ownership
- minimize irrelevant context loading
- use semantic filenames derived from the repo, such as `gateway-routing.instructions.md` or `runtime-packaging.instructions.md`

Do not assume the split must be `backend/plugins/platform` unless the analysis clearly shows that is the best fit.

---

#### **Repo-Wide Rules**

Keep this short. Include only cross-cutting rules such as:

- choose the owning subsystem before creating files
- avoid invented abstractions not present in the repository
- consult `./reference/api/*.yml` before finalizing external Insights Hub API integrations
- prefer package-local build and test commands for verification

### 2. Feature-Scoped Instruction Files

Generate 3 to 6 `.instructions.md` files under `{instruction_output_dir}`.

Each file must:

- use YAML frontmatter with a keyword-rich `description`, a `name`, and an `applyTo` string
- have a semantic filename derived from the inferred subsystem or domain
- group together categories, conventions, and constraints that are usually needed together during editing
- avoid becoming a second kitchen-sink document

Possible split signals include:

- separate top-level services with distinct ownership and conventions
- deployment or runtime assets that cut across multiple folders but share one operational concern
- plugin work that has materially different conventions from backend or infrastructure work
- local stubs or fixtures that behave differently from production services

### 3. Scoped Content Requirements

Across the scoped instruction files, cover every relevant category from `2-file-categorization.json`. Distribute categories to the file where they are most useful instead of duplicating the full catalog in every file.

For each scoped file:

- Explain what it is
- List 1–2 representative file examples
- Summarize key conventions based on its corresponding `5-style-guides/{category}.md`

---

### 4. Scoped Feature Scaffold Guidance

Within each scoped file, define how to plan and implement a new feature for that subsystem. Include:

- How to determine which categories of files to create
- Where to place those files
- How to follow naming and structure conventions
- Example: what files to create for a new component, a hook, or an API integration

This section should refer to actual conventions in the project and stay limited to what is present in that subsystem.

---

### 5. Integration Rules

From `3-architectural-domains.json`, summarize constraints in the scoped file where they are most relevant, such as:

- "All canvas logic must use `useCanvas`"
- "Components must use shared tokens from the design-system"
- "API requests go through `apiClient.ts` pattern"

This prevents LLMs from generating non-compliant or inconsistent files.

Additionally, include an explicit rule for external Insights Hub API calls in every scoped file that describes API integration behavior:

- When a generated feature calls external Insights Hub APIs, instruct the assistant to consult `./reference/api/*.yml` for endpoint paths, request/response field names, and payload shape examples before finalizing code.
- Do not invent endpoint paths or payload contracts when a relevant API example exists in `./reference/api/`.

---

### 6. Example Prompt Usage

Show how a user could prompt Copilot with a request that fits that subsystem and have it respond with the expected file set.

Use examples that match the inferred split. Cover every major instruction file with at least one example.

Only use categories and file types present in this project.

---

## ⚠️ Requirements

- **Do not** include invented best practices
- **Do not** list categories or conventions that aren’t supported by the codebase
- **Do not** omit any categories or domains defined in the analysis; distribute them across the split outputs
- **Do not** let `{workspace_instruction_file}` grow into a kitchen-sink document
- **Do not** hard-code a generic split when the repository analysis supports a more accurate boundary
- **Do not** omit the external Insights Hub API reference rule when API integration behavior is described

These files must give future LLMs enough information to build new features entirely within project conventions while loading only the subsystem guidance they need.

To clarify further, if any output file already exists, overwrite it.
