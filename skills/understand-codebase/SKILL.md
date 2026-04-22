---
name: understand-codebase
description: Understand the codebase and generate project-specific Copilot instructions by reviewing the prompt chain
---


{output_folder} = .results
{workspace_instruction_file} = /.github/copilot-instructions.md
{instruction_output_dir} = /.github/instructions

You are assisting with generating a slim {workspace_instruction_file} file plus feature-scoped instruction files under {instruction_output_dir} using a multi-step prompt chain.


## Execution Model

Use a coordinator pattern for this skill:

- The main agent owns prompt-chain planning, shared inputs, output schemas, merge decisions, completeness checks, and all final file writes.
- Subagents may be used for read-only repository analysis when a step is expensive, repetitive, or naturally decomposes by subsystem or domain.
- Treat subagent output as draft analysis, not final truth. The main agent must reconcile overlaps, resolve inconsistencies, and decide the final output.
- The main agent must use subagents for steps 1 through 4 unless the repository is trivially small or the user explicitly requests single-agent execution. Steps 5 and 6 must remain main-agent synthesis stages.
- The main agent must proactively maximize concurrency for steps 1 through 4. Do not let a single subagent walk the chain serially when the work can be decomposed and launched in parallel.

### Parallelization Rules

- Before launching any subagent, the main agent must review the full prompt chain in `./reference` so downstream requirements inform upstream execution.
- Subagents used for this skill should stay read-only. They may inspect files and summarize findings, but they should not write the final outputs for this workflow.
- Step 1 should be delegated to one subagent and launched immediately after prompt review.
- Step 2 must be split across multiple subagents by top-level folder, package, subsystem, or other clear ownership boundary for any non-trivial repository, and those subagents should be launched in parallel with step 1.
- Step 3 should be launched in parallel with steps 1 and 2 for non-trivial repositories. If step-1 or step-2 artifacts are not available yet, the subagent should infer a provisional domain model directly from the repository and note which assumptions should later be reconciled against `.results/1-techstack.md` and `.results/2-file-categorization.json`.
- Step 4 must also be launchable in the initial parallel wave. When step-3 domains are not available yet, the main agent should seed step-4 subagents with provisional domain or subsystem hypotheses based on prompt review and repository structure, and later reconcile those drafts against the final step-3 domain model.
- Step 5 should be executed by the main agent after step 2 is finalized, because it must cover every category with no omissions.
- Step 6 should be executed by the main agent after steps 1, 2, 3, and 5 are finalized, because it requires cross-step synthesis and final instruction-file generation.
- The main agent must not serialize steps 1 through 4 merely because the prompt files are numbered. Numbering defines output reconciliation order, not a requirement that a single worker execute the entire chain end-to-end.
- Subagents for steps 1 through 4 must return their findings to the main agent and stop. They must not autonomously continue into the next numbered prompt unless the main agent explicitly relaunches them for that specific step.

### Main-Agent Quality Gates

The main agent must enforce all of the following before moving to the next stage:

- A shared exclusion policy for generated files, dependency trees, vendored artifacts, and built outputs that should not be treated as authored source.
- A merged and deduplicated step-2 categorization that covers all relevant authored files, or explicitly explains each exclusion.
- A step-3 domain model that is based on concrete code evidence rather than generic architectural templates.
- A step-4 output set where every domain identified in step 3 has a corresponding deep-dive artifact.
- A step-5 output set where every category identified in step 2 has a corresponding style guide.
- A final step-6 output where `{workspace_instruction_file}` stays slim and subsystem detail is pushed into scoped instruction files under `{instruction_output_dir}`.


1. Navigate to the `./reference` folder
2. Review all the prompt files in this folder WITHOUT executing them.
    - This will help you understand the full scope of the prompt chain.
3. Review API specification files under `./reference/api/` before step 6.
    - Treat these files as source-of-truth examples for external Insights Hub API endpoint shapes and payload structures.
    - Use them when generating repository instructions that mention external API calls.
4. Confirm you have a full understanding of the prompt chain sequence.
5. Once you're familiar with the flow, begin executing the prompts in numerical order:
    - 1-determine-techstack.md
    - 2-categorize-files.md
    - 3-identify-architecture.md
    - 4-domain-deep-dive.md
    - 5-styleguide-generation.md
    - 6-build-instructions.md
6. For each step, output results into a corresponding `{output_folder}/` folder.
    - Mirror the step’s filename e.g., `1-determine-techstack.md` > `{output_folder}/1-determine-techstack.md`.

## Recommended Orchestration

Use this default execution order unless the user asks for a different strategy:

1. Main agent reviews all prompts under `./reference` and any API specs under `./reference/api/`.
2. Launch steps 1, 2, 3, and 4 in parallel.
3. For step 2, split by ownership boundary and merge the inventories in the main agent after the subagents return.
4. For step 3, allow the subagent to infer a provisional domain model directly from repository evidence, then reconcile it against the final step-1 and step-2 outputs in the main agent.
5. For step 4, launch one subagent per provisional domain or subsystem hypothesis, then normalize and remap those drafts in the main agent once step 3 is finalized.
6. Execute step 5 in the main agent after finalizing step 2.
7. Execute step 6 in the main agent after finalizing steps 1, 2, 3, and 5.

In practical terms, the expected concurrency pattern is:

- Wave 1: steps 1, 2, 3, and 4 all launch immediately.
- Wave 1a: step 2 is internally parallelized by ownership boundary.
- Wave 1b: step 4 is internally parallelized by provisional domain or subsystem hypothesis.
- Reconciliation: the main agent waits for the parallel wave to finish, then merges `.results` artifacts and resolves mismatches before steps 5 and 6.

This is the required default behavior for non-trivial repositories when the user requests maximum concurrency.

If any existing generated output files are present, overwrite them rather than treating them as source-of-truth inputs unless the user explicitly says to preserve them.

If the main agent chooses not to use subagents for steps 1 through 4, it must explicitly justify that choice using one of the allowed exceptions: the repository is trivially small, or the user explicitly requested single-agent execution.

Stop ONLY when:
    - All `instruction-generation` steps are complete
    - A slim `{workspace_instruction_file}` can be generated
    - Feature-scoped instruction files can be generated under `{instruction_output_dir}` using repository-specific domain boundaries inferred from the analysis.
