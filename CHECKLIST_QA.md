# QA checklist

- Confirm the target PrestaShop track and declared PHP compatibility.
- Confirm entrypoints stay thin and new business logic is in focused, testable classes.
- Run PHP syntax checks and Composer validation/autoload generation where applicable.
- Run the module's lint/style checks and automated tests.
- Require tests for every new module and focused or regression tests for changed business behavior.
- If installation, schema, configuration, hooks, translations, or multistore behavior changed, verify the relevant install/upgrade and runtime paths.
- Confirm English source strings and matching `cs-CZ` and `sk-SK` XLF entries.
- Report checks that ran and checks that could not run; do not describe inspection as execution.
- Verify no unrelated runtime, configuration, schema, or documentation changes.

