# CONVENTIONS

## DI and container access
- Prefer service wiring via module YAML configuration.
- Keep front-office container retrieval compatibility when legacy controllers use runtime lookup patterns (`->get(`, `module->get`, `this->get`, `container->get`, `getInstance()->get`).

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

