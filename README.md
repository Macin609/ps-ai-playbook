# ps-ai-playbook

Global repository for reusable PrestaShop AI conventions, checklists, and templates.

## Scope

- Cross-module standards only.
- No module-specific implementation memory.

## Consumption order (from a module workspace)

1. `ps-ai-playbook` (`<PLAYBOOK_ROOT>`)
2. Module AI docs (`<MODULE_AI_DOCS>`)
3. Module ADRs (`<MODULE_ADR_DOCS>`)

## Core files

- `CONVENTIONS.md`: normative Avalanche media module-development rules.
- `CHECKLIST_QA.md`: minimum verification before handoff.
- `CHECKLIST_SECURITY.md`: security review prompts.
- `CHECKLIST_LEGAL.md`: legal and confidentiality prompts.

## Connecting agents to this playbook

Keep reusable development rules in `CONVENTIONS.md` and their handoff checks in the appropriate checklist. Keep module-specific exceptions in module AI docs or ADRs, with an explicit rationale.

In each consuming workspace, use the agent's automatically loaded project instructions (for Codex, `AGENTS.md`) to require reading this playbook. Resolve `<PLAYBOOK_ROOT>` to the actual accessible location in that workspace; the placeholder is not automatically expanded. A separate playbook repository or a README link alone does not ensure that an agent loads the rules.

Suggested instruction for the consuming project's `AGENTS.md`:

```text
Before any PrestaShop module task, read <PLAYBOOK_ROOT>/CONVENTIONS.md,
then the module's docs/ai/ and relevant docs/adr/ records.
Apply these rules during implementation. Before handoff, work through
<PLAYBOOK_ROOT>/CHECKLIST_QA.md and the relevant security/legal checklists.
If the playbook is inaccessible, report that before implementation.
```

Keep the full rules in this playbook rather than copying them into every project's agent instructions, so updates have one authoritative source.

