# CONVENTIONS

## Canonical placeholders
- `<WORKSPACE_SHOP_ROOT>` = the shop folder in VS Code workspace (variable; never hardcode)
- `<PLAYBOOK_ROOT>` = `ps-ai-playbook`
- `<MODULE_ROOT>` = `modules/<module_name>/`
- `<MODULE_DOCS>` = `<MODULE_ROOT>/docs/`
- `<MODULE_AI_DOCS>` = `<MODULE_DOCS>/ai/`
- `<MODULE_ADR_DOCS>` = `<MODULE_DOCS>/adr/`
- `<MODULE_CONFIG>` = `<MODULE_ROOT>/config/`
- `<MODULE_SRC>` = `<MODULE_ROOT>/src/`
- `<MODULE_FRONT>` = `<MODULE_ROOT>/controllers/front/`
- `<MODULE_ADMIN>` = `<MODULE_ROOT>/controllers/admin/`
- `<MODULE_HOOK_FRONT>` = `<MODULE_ROOT>/src/Module/Hook/Front/`

## Structure baseline
- Keep module memory in `<MODULE_AI_DOCS>` and module ADRs in `<MODULE_ADR_DOCS>`.
- Keep module runtime code and configuration outside this playbook.
- Use placeholders in docs; avoid hardcoding shop folder names.

## DI and container access
- Prefer service wiring via module YAML configuration.
- Keep front-office container retrieval compatibility when legacy controllers use runtime lookup patterns (`->get(`, `module->get`, `this->get`, `container->get`, `getInstance()->get`).
- For FO visibility audits, scan both `<MODULE_FRONT>` and `<MODULE_HOOK_FRONT>`.
- Expose required FO service IDs via `<MODULE_CONFIG>/front/services.yml`.
- Do not broaden defaults in `<MODULE_CONFIG>/common.yml` for hotfixes.

## Configuration and installer baseline
- Keep typed configuration keys in module config data classes.
- Prefer installer-managed defaults and SQL install/uninstall lifecycle.
- Treat these as module runtime concerns; document behavior, do not mutate in docs-only tasks.

## Persistence and operations baseline
- Prefer repository-layer DB access and structured logging.
- Idempotent cron/pairing flows should use deterministic dedupe keys and locking where available.

## Security baseline
- OAuth callbacks must validate and consume one-time state.
- Sensitive credentials/tokens must be encrypted at rest.
- Cron endpoints should require a secret token.

## Error-handling baseline
- Classify integration errors consistently (for example: `misconfig`, `transient`, `needs_reauth`).
- Prefer structured logging with context and payload.

## Runtime-change guardrails for hotfix tasks
- Do not change PHP runtime behavior unless explicitly requested.
- Do not broaden DI visibility defaults globally for quick fixes; use minimal scoped exposure.

