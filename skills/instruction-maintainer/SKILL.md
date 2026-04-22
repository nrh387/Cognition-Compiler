---
name: instruction-maintainer
description: Detect reusable rules in user corrections or manual rewrites of AI output, ask whether they should be persisted into copilot-instructions or scoped instructions, and maintain them with minimal changes.
---

[Task]
    Identify reusable rules when the user corrects AI behavior.
    If the correction reflects a stable rule rather than a one-off preference, first ask whether it should be written into the instructions, then persist it to the smallest correct location in `.github/copilot-instructions.md` or `.github/instructions/*.instructions.md`.

[Dependency Check]
    Run this check first when the skill starts.

    Required:
    - `.github/copilot-instructions.md` -> if missing, stop and ask the user to confirm the repository-level instruction entry point first
    - `.github/instructions/` or an equivalent scoped instruction directory -> if missing, degrade to maintaining only `.github/copilot-instructions.md`
    - Current correction context -> at least one of the following must be available: the user's exact wording, the user's manual edit, the AI output being corrected, or the relevant file/code snippet

    Optional:
    - `git diff` or editor changes -> used to compare what the user manually corrected
    - Historical instruction-generation context -> used to determine whether an equivalent rule is already covered by another scoped instruction

    Installation strategy:
    - This skill requires no additional installation

[First Principles]
    **Confirm before persisting**: Any new rule extracted from a user correction must be confirmed with the user before it is written into instructions.
    **Prefer rules over anecdotes**: Persist only reusable, portable, and verifiable behavior constraints, not one-off requests or temporary tastes.
    **Write to the smallest correct scope**: If a rule belongs in a scoped instruction, do not bloat the repository-level instruction file; if a local rule is sufficient, do not write vague global guidance.
    **Extract rules from evidence**: Prefer the user's wording, the user's edit diff, and code ownership boundaries as evidence; do not expand based on guesswork.
    **Use external docs only when needed**: Check documentation first only when the correction depends on current external platform, framework, or API constraints.

[File Structure]
    ```
    instruction-maintainer/
    └── SKILL.md
    ```

[Output Style]
    **Tone**:
    - Direct, short, and focused on confirmation and execution
    - State the candidate rule first, then the destination, then ask whether it should be written

    **Principles**:
    - x Do not change instruction files without telling the user
    - x Do not turn high-guessing statements like “I think you mean...” into rules
    - v State the extracted rule explicitly
    - v State which instruction file it should go into and why

    **Typical phrasing**:
    - "I extracted a candidate rule from this correction: .... If you want this to be the default behavior going forward, I can add it to the instructions."
    - "This constraint looks like a `plugins/`-local rule rather than a repository-wide one. Should I add it to the corresponding scoped instruction file?"

[Trigger Signals]
    Enter the rule-extraction flow when any of these signals appears:
    - The user explicitly rejects AI behavior, such as “that’s not right”, “don’t do it like that”, “you got it wrong”, or “don’t default to that”
    - The user points out a rule conflict, such as “this repo is not organized that way”, “this must go through the existing abstraction”, or “you can’t change this layer”
    - The user manually rewrites AI output, and the change is not just formatting but changes structure, ownership, process, naming, validation, or boundary handling
    - The user corrects the AI more than once on the same kind of issue, indicating a stable pattern rather than an isolated preference

[Rule Extraction Checklist]
    Must determine:
    - Whether the correction changes a behavior rule, code-ownership rule, validation rule, or just a one-off task instruction
    - Whether the rule applies at repository, subsystem, directory, file-type, or current-task scope
    - Whether an equivalent rule already exists in the repository and the AI simply failed to follow it
    - Whether the rule can be written as a positive, actionable, and verifiable sentence

    Recommended:
    - Whether there is a concrete counterexample showing what goes wrong if the rule is not followed
    - Whether there is a better owning instruction file than continuing to grow `.github/copilot-instructions.md`

    Should not be persisted:
    - Aesthetic preferences or temporary wording from the current turn
    - Guesses based on unconfirmed facts
    - Information that duplicates existing instructions without adding any new constraint

[Rule Persistence Strategy]
    **Extract first, confirm second**: Restate the correction as a single candidate rule, then ask whether it should be written into instructions.

    **Patch gaps instead of duplicating**: If an existing instruction already covers the rule, do not add a synonym; prefer rewriting it into a clearer and more actionable form.

    **Place rules by ownership**:
    - Cross-repository entry points, global principles, and routing/selection rules -> `.github/copilot-instructions.md`
    - Subsystem-specific constraints, directory ownership, testing/validation expectations, and abstraction boundaries -> the matching `.github/instructions/*.instructions.md`
    - Rules that apply only to the current task -> do not write them into instructions; only follow them in the current conversation

    **Keep changes minimal**: Add only one rule, or one tightly related group of rules, at a time. Do not rewrite the whole instruction set opportunistically.

    **Write corrections as operating rules**: Prefer “what to do / what not to do / when it applies” phrasing instead of recording raw conversation text.

[Information Sufficiency Check]
    Must satisfy:
    - It can identify which AI behavior was corrected
    - It can extract a candidate rule that still makes sense outside the current conversation
    - It can explain which instruction file the rule belongs in

    Preferably satisfy:
    - The user's wording or manual diff is available as evidence
    - Code ownership or directory boundaries support the placement decision

    If not satisfied:
    - Do not write any instruction
    - Only restate the observed correction to the user and ask for the missing information

[Workflow]
    [Step 1: Capture The Correction]
        Gather the context of the most recent explicit user correction, rejection, redo, or manual modification of AI output.
        If the user edited the output manually, compare the before/after difference first to identify what behavior changed.

    [Step 2: Decide Whether It Should Persist]
        Use [Rule Extraction Checklist] to determine whether this is a stable rule.
        If it is only a one-off preference, temporary request, or unsupported guess, stop here and do not write any instruction.

    [Step 3: Generate A Candidate Rule]
        Extract the correction into one candidate rule.
        The candidate rule must make its scope, action, and applicability clear, and it must be directly usable in future tasks.

    [Step 4: Ask The User For Confirmation]
        Tell the user clearly:
        - what correction was observed
        - what candidate rule was extracted
        - which instruction file it should go into and why
        Ask one explicit question: should this rule be written into the instructions?

    [Step 5: Update The Instructions]
        After user confirmation, update the smallest correct owning instruction file.
        If the rule already exists but is poorly expressed, make the smallest rewrite instead of adding a duplicate section.

    [Step 6: Report The Result]
        Tell the user what rule was written, where it was written, and what default AI behavior will change because of it.

[Initialization]
    Execute [Step 1: Capture The Correction]
