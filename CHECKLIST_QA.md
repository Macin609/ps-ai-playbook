# QA checklist

- Confirm the target PrestaShop track and declared PHP compatibility.
- Confirm entrypoints stay thin and new business logic is in focused, testable classes.
- For service wiring changes, check placement and imports against `CONVENTIONS.md` (Hooks and dependency injection). Verify dependency resolution and the affected runtime paths in each intended admin/front container, including context-specific PrestaShop implementations; confirm context-only services are not registered in the opposite container. Report unavailable runtime checks explicitly.
- Confirm global identifiers are module-prefixed constants with one owner; configuration uses its typed façade and explicit scope.
- Confirm every admin route/action has the correct ACL, HTTP method, CSRF behavior, route constraints, and installed visible/hidden tab mapping.
- Run PHP syntax checks and Composer validation/autoload generation where applicable.
- Run the module's lint/style checks, Autoindex, and automated tests.
- Require tests for every new module and focused or regression tests for changed business behavior.
- If installation, schema, ObjectModel/ORM mapping, configuration, hooks, translations, or multistore behavior changed, verify the relevant install/upgrade and runtime paths.
- Confirm unique constraints/indexes match invariants and query paths; confirm no hardcoded shop/language/entity IDs.
- Confirm monetary decisions avoid floats and stored operational timestamps use UTC.
- Build and test the final production artifact after dependency scoping; confirm no development packages or obsolete Avalanche wrapper packages were added.
- For API/cron changes, verify timeouts, TLS, retry bounds, locks, idempotency, batching/deadlines, summaries, and operational state.
- Confirm English source strings and matching `cs-CZ` and `sk-SK` XLF entries.
- Report checks that ran and checks that could not run; do not describe inspection as execution.
- Verify no unrelated runtime, configuration, schema, or documentation changes.
