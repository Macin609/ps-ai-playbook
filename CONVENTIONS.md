# Avalanche media PrestaShop module conventions

This file contains the cross-module rules that an agent should load for every PrestaShop module task. Module-specific documentation and ADRs override it only where they explicitly record a deliberate exception.

## Paths and scope

- `<WORKSPACE_SHOP_ROOT>`: the shop root in the current workspace; never hardcode it.
- `<PLAYBOOK_ROOT>`: this `ps-ai-playbook` repository.
- `<MODULE_ROOT>`: `modules/<module_name>/`.
- `<MODULE_SRC>`: `<MODULE_ROOT>/src/`.
- `<MODULE_CONFIG>`: `<MODULE_ROOT>/config/`.
- `<MODULE_AI_DOCS>`: `<MODULE_ROOT>/docs/ai/`.
- `<MODULE_ADR_DOCS>`: `<MODULE_ROOT>/docs/adr/`.
- Keep only reusable, cross-module rules here. Keep module knowledge and decisions in the module repository.

## Target platform first

- Before creating a module, establish whether it targets PrestaShop 1.7-8 or PrestaShop 9. These are separate module versions; do not attempt one implementation for both tracks.
- If the target is not stated, ask before implementation.
- Derive the supported PHP versions, PrestaShop APIs, Symfony APIs, and packaging requirements from the selected PrestaShop version and its official documentation.
- Declare the resulting compatibility in Composer and the module bootstrap. Do not use syntax or APIs outside the declared range.
- For an existing module, preserve its declared compatibility unless the task explicitly changes it.

## References and architecture

- Inspect relevant Avalanche media modules before designing a solution. Prefer current modules on the same PrestaShop track and use `am_bankcsasapi` and `am_limitorders` as general modern references. Treat older code as evidence of compatibility, not automatically as a pattern to copy.
- Use PSR-4 classes under `<MODULE_SRC>`. Choose feature-oriented or Domain/Application/Infrastructure folders according to complexity; a fixed directory taxonomy is not required.
- Keep the root module class, controllers, hooks, commands, and cron endpoints as thin entrypoints. They validate and normalize input, authorize the call, invoke an application service or handler, and format the result.
- Business rules, workflows, persistence, mapping, and API communication belong in focused classes. Do not add extensive inline business logic, god methods, or unrelated responsibilities to an existing large class.
- Prefer constructor injection and explicit interfaces at boundaries that need substitution in tests. Register classes with several dependencies as container services.

## Composer and packaging

- Use Composer PSR-4 autoloading for `<MODULE_SRC>` and classmap only legacy PrestaShop entrypoints that require it.
- Put test namespaces and test-only packages in `autoload-dev` and `require-dev`.
- The production module package must contain a generated `vendor/` when it has Composer dependencies. Load `vendor/autoload.php` from the module bootstrap and fail clearly if a required production autoloader is missing; do not maintain a second fallback autoloader.
- Keep production installs optimized and exclude development dependencies.

## Hooks and dependency injection

- Implement each non-trivial hook in a dedicated class under `src/Module/Hook/`, split into `Admin/` and `Front/` where useful. The root `hook...()` method only delegates and handles the PrestaShop boundary result/error.
- A small hook with only the module, context, and hook parameters may be instantiated by the delegating method. Register hooks with additional collaborators as services and inject their dependencies.
- Keep service visibility scoped. Expose a service publicly only when a legacy controller or hook must retrieve it at runtime; do not broaden defaults in `common.yml`.

## Installation, removal, and upgrades

- Keep install/uninstall orchestration in a dedicated installer class. Declare hooks, controllers or tabs, configuration defaults, mail templates, and SQL lifecycle there rather than scattering them through the root module class.
- Installation and upgrades must be repeatable and safe after partial failure. Check every operation, clean up partial installation where practical, and return a real failure to PrestaShop.
- Version schema and data changes through PrestaShop upgrade scripts. Do not silently modify an existing install schema without a matching upgrade path.
- Do not use `die()` or a successful HTTP response to report migration failure.
- Make uninstall behavior explicit. Remove module secrets and orphaned configuration; remove business data only when the module's agreed uninstall policy says it is destructive.

## Database and multistore

- Put database access behind repositories or other focused persistence classes. `Db`, `DbQuery`, ObjectModel, and Doctrine DBAL are all acceptable when appropriate to the selected PrestaShop API; no single database API is mandatory.
- Use `_DB_PREFIX_`, cast numeric identifiers, escape string values with the appropriate PrestaShop/DBAL mechanism, whitelist dynamic SQL fragments, and check write results.
- Use transactions and locking for multi-step writes, concurrency-sensitive jobs, and workflows that must remain consistent. Make retry and deduplication behavior explicit for cron and integration flows.
- Decide and document the scope of every configuration value and stored record: global, shop group, or shop. Use explicit shop constraints for configuration and persist/filter `id_shop` for shop-specific data. Never assume that credentials, tokens, or operational state share the same scope.

## Errors, logging, and public responses

- Catch failures at system boundaries where they can be logged, translated to the expected PrestaShop return value, or converted to an HTTP response. Do not silently discard failures inside business logic.
- Use structured logging with operation, severity, stable error code/category, relevant entity and shop IDs, and exception context. Never log secrets, authorization headers, raw tokens, or unfiltered sensitive payloads.
- Preserve exception chaining when converting errors. Distinguish configuration, validation, authorization, transient integration, and unexpected internal failures where callers act on them differently.
- Public responses must use stable status codes and generic messages. Do not expose exception messages, stack traces, SQL, filesystem paths, credentials, or upstream response bodies.

## Translations

- Write source translation strings in English and use the modern PrestaShop translation system with a consistent `Modules.<Module>.<Domain>` domain.
- Create and maintain Czech (`cs-CZ`) and Slovak (`sk-SK`) XLF translations as part of the same change. Keep placeholders identical across all locales.
- Provide both HTML and plain-text mail templates where the target PrestaShop mail system expects them, including the required language variants.
- Do not build user-facing sentences by concatenating translated fragments or exception text.

## Security

- Protect cron and machine endpoints with a non-empty, generated secret and compare it with `hash_equals`. Return an authorization failure before running business logic.
- Require the appropriate customer, employee, permission, object ownership, and shop-context checks for every controller action. Use PrestaShop/Symfony CSRF protection for state-changing browser requests.
- Validate and normalize all external input. Escape for the output context and use allowlists for action names, sort directions, filenames, routes, and other structural values.
- Validate and consume one-time OAuth state, protect callback flows against replay, encrypt sensitive credentials at rest, and redact them from logs and diagnostics.

## Tests and verification

- Every new module must include automated tests and a working documented test command. New or changed business logic must have focused tests; bug fixes should include a regression test.
- Isolate pure business rules from PrestaShop globals so they can be unit tested. Use narrow adapters, fixtures, or stubs for PrestaShop and database integration; add integration tests when behavior depends on schema, transactions, hooks, multistore, or framework wiring.
- Before handing off a change, run the checks relevant to the affected module: PHP syntax, Composer validation/autoload generation, style/lint, automated tests, install/upgrade path when changed, and targeted manual verification in the declared PrestaShop version.
- Report exactly which checks ran and any checks that could not run. Do not claim verification from inspection alone.

