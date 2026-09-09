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

## Identifiers and configuration

- Namespace routes, service IDs, configuration/cache/lock/cron keys, custom hooks, assets, and other global identifiers with the module name. Use structured service IDs such as `<module>.<area>.<responsibility>`.
- Give module name/version, controller and tab names, grid IDs, configuration keys, persistent state/error codes, and module-owned schema identifiers a canonical owner under the shared-contract rules below. Use named constants or an appropriate typed contract supported by the declared platform.
- Keep configuration keys in a central configuration data class. Access them through a typed configuration service that applies scope, defaults, normalization, decoding, and critical-value validation; business code must not use raw configuration-key strings.
- Give dynamic configuration keys a central factory/prefix and an explicit cleanup lifecycle. Treat the release version in Composer metadata, the root module class, and other required manifests as representations of one canonical value under the duplication rules below.

## Canonical sources for shared contracts

- Every technical or domain value whose meaning must stay consistent across multiple consumers must have one explicit owner and one canonical source. This includes minimum/maximum/default values; configuration names; table, column, primary-key and foreign-key names; state, error, reason and event codes; hook, route, service and integration-operation names; parser, fingerprint, schema and rules versions; form-field and request-parameter identifiers; module names; and shared timeout, batch and other operational values.
- Place the source close to the component that owns the concept. Use a named constant, enum or equivalent typed contract, immutable value object, schema/configuration definition, public integration contract, or method for a derived value, as appropriate to the declared platform. Do not create a global class of unrelated constants.
- Forms, validators, repositories, services, controllers, hooks, CLI/cron entrypoints, templates and other production consumers must obtain the value from that source. When direct access is unsuitable, pass it through a service, DTO, parameter or view model; do not create another manually synchronized definition for convenience.
- Share by meaning, not by matching text or numbers. Keep different concepts separate even when their current values match, such as `DEFAULT_BATCH_LIMIT` and `MAX_BATCH_LIMIT` both being `50`. A one-off local literal without contract meaning does not need a constant merely for uniformity.

### Required representations and independent modules

- When a platform or file format requires physical duplication, name the canonical source and treat other occurrences as derived or synchronized representations. Update them through the supported process, add an automated or explicit consistency check, and confirm their agreement at handoff. Regenerate generated files with the project's standard generator; do not hand-edit them.
- Do not access another module's internal constants if that creates an unapproved runtime dependency. Values shared between modules must be explicit integration contracts with a named owning module.
- When an independent consumer must mirror an integration value locally, document the intentional duplication and owner, require a contract test that checks both sides, and coordinate producer and consumer updates when the contract changes. Do not introduce a shared runtime package solely to remove such a mirror; follow the existing shared-package criteria.

### Changing a shared contract

- Before introducing or changing a contract, identify its owner and canonical source, then search the entire affected scope for both old and new values and their symbolic references. Include production code, configuration, schemas, integrations, tests, documentation and generated artifacts.
- Replace unjustified production copies with references to the source. Classify each retained occurrence as an independent test expectation, documentation, generated output, a required platform representation, a documented integration mirror, a versioned historical contract, or a genuinely different concept. Explain why retained production copies are necessary.
- Update affected documentation and supported generated outputs, verify all required representations, and run the relevant contract and behavior checks. At handoff, name the canonical source, report the scope checked and any limitations, and confirm whether any unjustified production duplication remains. Repeated manual edits to independent production definitions indicate missing or incorrectly scoped ownership.

## Composer and packaging

- Use Composer PSR-4 autoloading for `<MODULE_SRC>` and classmap only legacy PrestaShop entrypoints that require it.
- Put test namespaces and test-only packages in `autoload-dev` and `require-dev`.
- The production module package must contain a generated `vendor/` when it has Composer dependencies. Load `vendor/autoload.php` from the module bootstrap and fail clearly if a required production autoloader is missing; do not maintain a second fallback autoloader.
- New modules must not depend on `avalanchemedia/prestashop-module-configuration`, `avalanchemedia/prestashop-module-hooks`, or `avalanchemedia/prestashop-module-installer`. Implement only the required installer, hook, and configuration adapters locally for the selected PrestaShop track. Do not remove these packages from existing modules without an explicit migration task.
- Prefer small module-local composition over copied universal base classes. Consider a shared runtime package only after the same substantial, stable behavior is used by at least three modules and has a versioned API plus CI for every supported PrestaShop track.
- Scope/prefix every third-party production dependency bundled with a module into the module namespace unless the selected PrestaShop version explicitly provides and supports that library. Exclude Composer/build plugins and development-only packages from the scoped production artifact.
- Keep production installs authoritative and optimized, use `prepend-autoloader: false` where supported, and exclude development dependencies. Test the final scoped production artifact, not only the unscoped source tree.
- Run PrestaShop Autoindex after adding or moving distributable directories. Include generated protective `index.php` files and do not hand-edit them.

## Admin controllers and routes

- Every back-office action, including AJAX actions, must declare authorization through the mechanism supported by the selected PrestaShop track. Map read/create/update/delete operations to the matching permission; securing only the menu or parent controller is insufficient.
- Bind each admin route to the correct installed tab/ACL identifier (`_legacy_controller` and `_legacy_link` where the track requires them). Install a hidden tab when an action needs permissions but no visible menu item.
- Use `GET` only for safe reads. State-changing actions use `POST` or another supported mutation method, validate CSRF, constrain route identifiers (for example to digits), and redirect after successful form writes.
- Prefer framework response objects such as `JsonResponse`. A legacy controller may use one shared response helper; do not scatter `die(json_encode(...))` across actions.

## Hooks and dependency injection

- Implement each non-trivial hook in a dedicated class under `src/Module/Hook/`, split into `Admin/` and `Front/` where useful. The root `hook...()` method only delegates and handles the PrestaShop boundary result/error.
- A small hook with only the module, context, and hook parameters may be instantiated by the delegating method. Register hooks with additional collaborators as services and inject their dependencies.
- Give every hook an explicit input and return contract. On a caught boundary failure, return the neutral value required by that hook; do not interchange `false`, `''`, `[]`, and `void` without regard to the contract.
- Before registering or changing a service, identify whether it is used by admin, front, or both. Register shared services in `<MODULE_CONFIG>/common.yml`, admin-only services in `<MODULE_CONFIG>/admin/services.yml`, and front-only services in `<MODULE_CONFIG>/front/services.yml`. Keep services used exclusively by one side out of the other side's container.
- Inspect configuration imports and automatic service discovery as well as individual definitions. Ensure shared definitions are loaded by both containers without importing admin-only definitions into front or front-only definitions into admin.
- Before injecting collaborators into a new or changed service, verify their service IDs, aliases, implementations, and transitive dependencies in each intended container for the declared PrestaShop version. Some PrestaShop services differ between admin and front; availability or behavior in one container does not establish availability or equivalent behavior in the other.
- A shared service must have a valid dependency graph in both containers. When a collaborator differs by context, use an explicit module-owned interface with context-specific wiring or adapters in the admin/front configuration. Do not move a context-only dependency into `common.yml` merely to make injection resolve.
- Keep service visibility scoped. Expose a service publicly only when a legacy controller or hook must retrieve it at runtime; do not broaden defaults in `common.yml`.

## Installation, removal, and upgrades

- Keep install/uninstall orchestration in a dedicated installer class. Declare hooks, controllers or tabs, configuration defaults, mail templates, and SQL lifecycle there rather than scattering them through the root module class.
- Build the module-local installer from only the capabilities the module needs: schema/data, hooks, scoped configuration, visible or ACL-only tabs, mail templates, and front-controller metadata. Do not copy a universal installer wrapper wholesale.
- Installation and upgrades must be repeatable and safe after partial failure. Check every operation, clean up partial installation where practical, and return a real failure to PrestaShop.
- Version schema and data changes through PrestaShop upgrade scripts. Do not silently modify an existing install schema without a matching upgrade path.
- Do not use `die()` or a successful HTTP response to report migration failure.
- Make uninstall behavior explicit. Remove module secrets and orphaned configuration; remove business data only when the module's agreed uninstall policy says it is destructive.

## Database and multistore

- Put database access behind repositories or other focused persistence classes. `Db`, `DbQuery`, ObjectModel, and Doctrine DBAL are all acceptable when appropriate to the selected PrestaShop API; no single database API is mandatory.
- Define module-owned table names without `_DB_PREFIX_`. If an ObjectModel owns the mapping, keep `TABLE_NAME` and `PRIMARY_KEY` on that model and use them in `$definition`; if only one repository consumes an identifier, use a private repository constant. When installation, repositories, diagnostics or other infrastructure consumers share schema identifiers (including columns and foreign keys), reuse the mapping owner or a small Infrastructure/Database schema metadata class. Domain classes must not know table names.
- Keep SQL schema and ObjectModel/ORM mapping aligned, with upgrade scripts implementing the intended schema transitions. ObjectModel fields must declare correct types, validators, size, nullability, and required state. Where DDL/PHP duplication is required at the install boundary, identify the canonical schema owner and check consistency under the shared-contract rules; cover critical mapping with a schema self-check or test.
- Preserve version-specific identifiers and values required by historical migrations. Do not replace them with a mutable current-schema definition if that changes an older upgrade's meaning. Treat them as explicit historical contracts and verify that the upgrade path reaches the intended current schema.
- Use `_DB_PREFIX_`, cast numeric identifiers, escape string values with the appropriate PrestaShop/DBAL mechanism, whitelist dynamic SQL fragments, and check write results.
- Encode identity and idempotency in `UNIQUE` constraints where possible, and index real join/filter/queue/cron paths, including their shop scope. New tables use the target-track engine placeholder and `utf8mb4` unless its documentation requires otherwise.
- Use transactions and locking for multi-step writes, concurrency-sensitive jobs, and workflows that must remain consistent. Make retry and deduplication behavior explicit for cron and integration flows.
- Decide and document the scope of every configuration value and stored record: global, shop group, or shop. Use explicit shop constraints for configuration and persist/filter `id_shop` for shop-specific data. Never assume that credentials, tokens, or operational state share the same scope.
- Never hardcode database IDs for shops, languages, order states, or other installed entities. Resolve languages by ISO code and system entities by context, stable reference, or configuration; seed data must not depend on installation order.

## Money and time

- Store money in `DECIMAL` columns and carry it through domain/application code as normalized decimal strings or a money value object. Perform arithmetic and comparisons through one decimal service with explicit scale; do not use binary floats for price/payment decisions.
- Store operational timestamps in UTC and convert to the relevant shop/admin timezone only for presentation. Name timestamps by meaning (attempted, succeeded, processed, expires) and isolate or inject the clock for testable time-dependent rules.

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

## External integrations and cron

- HTTP clients must set connect/request timeouts, verify TLS, preserve large numeric identifiers, classify failures, and sanitize response data before it reaches exceptions or logs. Propagate a correlation ID where the upstream API supports it.
- Retry only idempotent operations and transient failures. Use a bounded exponential backoff with jitter, respect `Retry-After`, and reschedule instead of blocking a request worker for a long server-advertised delay.
- Cron/integration jobs must use a lock, bounded batches or a deadline, deterministic idempotency, and a machine-readable summary. Track last attempt, last success, and last error separately.
- Integration modules should expose a read-only self-check for critical configuration, schema, and runtime dependencies without revealing secret values.

## Tests and verification

- Every new module must include automated tests and a working documented test command. New or changed business logic must have focused tests; bug fixes should include a regression test.
- Isolate pure business rules from PrestaShop globals so they can be unit tested. Use narrow adapters, fixtures, or stubs for PrestaShop and database integration; add integration tests when behavior depends on schema, transactions, hooks, multistore, or framework wiring.
- Contract tests may assert explicit expected literals as independent checks. Do not derive every expectation from the same production constant when that would allow an incorrect contract change to pass undetected. For intentionally mirrored cross-module contracts, verify producer/consumer agreement as well as the expected contract where relevant.
- Before handing off a change, run the checks relevant to the affected module: PHP syntax, Composer validation/autoload generation, style/lint, Autoindex, automated tests, final scoped production build, install/upgrade path when changed, and targeted manual verification in the declared PrestaShop version.
- Report exactly which checks ran and any checks that could not run. Do not claim verification from inspection alone.
