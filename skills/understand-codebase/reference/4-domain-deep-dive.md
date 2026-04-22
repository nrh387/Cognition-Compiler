Execution requirement:

- This step must be executed with subagents unless the repository is trivially small or the user explicitly requests single-agent execution.
- When subagents are used, prefer one subagent per architectural domain from `./{output-folder}/3-architectural-domains.json` when that file already exists. Otherwise, use one subagent per provisional domain or subsystem hypothesis supplied by the main agent.
- Subagents should stay read-only and return draft domain writeups with concrete examples.
- The main agent remains responsible for normalizing structure across domains, checking that every step-3 domain is covered, and writing the final files under `./{output-folder}/4-domains/`.
- Step-4 domain subagents are expected to run in parallel with one another and may launch immediately when the user requests maximum concurrency, even if step 3 has not finished yet.
- If the final step-3 domain list is not available at launch time, treat the provided `{domain}` value as a provisional hypothesis to explore rather than a guaranteed final domain.
- After returning findings for this step, stop and hand control back to the main agent. Do not autonomously continue to later prompts.

> This task may take some time — that is expected and acceptable.
>
> Do **not** skip files or produce partial results due to time or complexity. Accuracy and completeness are **mission-critical**.
>
> You are permitted to take as long as necessary to:
>
> - Review every relevant file
> - Extract actual patterns and conventions
> - Produce complete, high-fidelity output
>
> Do not optimize for speed or brevity. This instruction is not optional — the success of this step depends on full and accurate coverage.

For each of the domains listed in ./{output-folder}/3-architectural-domains.json, or for each provisional domain hypothesis supplied by the main agent when that file is not available yet, you're analyzing the codebase to understand how it implements the architectural domain: "{domain}".

If available, the tech stack is summarized in ./{output-folder}/1-techstack.md. If not, infer what you need directly from the repository.

Your Task:

- Examine relevant files for this domain.
- Identify consistent patterns, tools, conventions, and practices.
- Include real code examples that reflect actual usage in the codebase.
- Focus on what the project is doing — not what it should do.

Write your findings to: `./{output-folder}/4-domains/{domain}.md`

If the domain turns out to be misnamed, overlapping, or only partially supported by the repository, say so explicitly in the writeup rather than blocking execution.

Requirements:

- If code snippets are included, use exact code examples from project files.
- Do not invent recommendations or include external best practices.
- Only describe what is actually used in this project.

Your goal is to document how the "{domain}" domain is implemented within this specific codebase in such a way that anyone could leverage or add features to it.
