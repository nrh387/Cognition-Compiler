Execution requirement:

- This step must be executed with subagents unless the repository is trivially small or the user explicitly requests single-agent execution.
- When subagents are used, split the analysis by clear ownership boundaries such as top-level folders, packages, or subsystems.
- Subagents should stay read-only and return draft inventories or categorizations to the main agent.
- The main agent remains responsible for merge decisions, deduplication, exclusion policy, completeness validation, and writing the final `./{output-folder}/2-file-categorization.json` artifact.
- Step-2 subagents are expected to run in parallel with step 1 and with each other.
- After returning findings for this step, stop and hand control back to the main agent. Do not autonomously continue to the next prompt.

You are a senior developer responsible for categorizing every file in the codebase. If ./{output-folder}/1-techstack.md already exists, use it as supporting context. If it does not exist yet, infer what you need directly from the repository and continue without blocking on step 1.

Your task:

- Visit every file in the codebase. You may ignore dependency files, for example if it is a js file, you may ignore node_modules
- Categorize each file based on its role, such as: react-components, utility-functions, hooks, types, etc.

Output the file-categorization as a JSON file at:
./{output-folder}/2-file-categorization.json

```json
{
  "react-components": ["./src/components/Button.tsx"],
  "hooks": ["./src/hooks/useUser.ts"]
}
```

A single file can appear in multiple categories if appropriate.

> This task may take some time — that is expected and acceptable.
> Do **not** skip files or produce partial results due to time or complexity. Accuracy and completeness are **mission-critical**.
> If a file is listed in ./{output-folder}/2-file-categorization.json or is part of a relevant domain, it **must** be included in your analysis.
> Do not optimize for speed or brevity. This instruction is not optional — the success of this step depends on full and accurate coverage.

You are permitted to take as long as necessary to:

- Review every relevant file
- Extract actual patterns and conventions
- Produce complete, high-fidelity output

After writing ./{output-folder}/2-file-categorization.json, return the categorization to the main agent for merge and orchestration.
