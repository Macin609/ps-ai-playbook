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

## Human-readable source and formatting

- Write code for a developer debugging it manually: descriptive names, focused functions, explicit control flow and error handling. Avoid compressed multi-statement lines, deeply nested expressions and clever shortcuts that hide intent. Comments explain reasons, invariants or compatibility constraints rather than restating obvious operations.
- At project intake, inspect existing formatting/linting conventions and supported language versions. Preserve a compatible established style; otherwise configure a project-local formatter and a non-mutating check before feature work. Separate broad initial normalization from behavior changes in its own commit or PR.
- Use Prettier for the project's supported JavaScript/TypeScript, CSS/SCSS and hand-maintained JSON/YAML. Documentation/HTML can opt in after checking their syntax and generation contracts. Keep PHP under PHP-CS-Fixer with the project's PSR-12-compatible rules and declared minimum PHP; do not run two competing formatters on one file.
- Do not parse Twig or Smarty as plain HTML. Use a formatter/plugin only after verifying the exact syntax and supported platform; otherwise retain explicit readability and rendering review. Choose a suitable existing formatter for Python and other languages rather than forcing them through Prettier. Record gaps in automated coverage.
- Commit formatter configuration, explicit scope/exclusions, exact tool versions and dependency lockfiles, documented format/check commands, and an EditorConfig appropriate to the project. Editors, agents and CI use the same local tools. Do not depend on an unpinned global formatter or install tools into a client's shop.
- Exclude dependencies, minified/build outputs, binaries, package-manager lockfiles and historical evidence. Generate derived resources through their canonical owners; do not independently reformat generated translations, mail templates, wiring or manifests. Explain exceptions; never exclude a failing source merely to obtain a green check.
- After editing, format owned sources, inspect the diff, regenerate affected outputs, then run format checks, lint, contract checks and relevant tests before independent review. CI must check without writing and block build/release on failure. Missing tools or unexecuted checks are reported as such, not passed; do not bypass sandbox restrictions to install them.
- Formatter output is not evidence of correctness, security or comprehensibility. Reviewers also assess naming, responsibilities and readable error paths. Distinguish style suggestions from security vulnerabilities.
- Keep readable canonical sources in Git. Minification is a separate optional build step. Formatting tools, node_modules, their development configuration and instructions must stay outside the installable ZIP under the runtime package allowlist.

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

## Licensing and provenance

- Preserve the project's declared license and intended distribution model. Identify whether work is used internally, supplied to clients, redistributed as a ZIP/source, or provided over a network. Do not relicense proprietary code or accept new contractual obligations without the owner's instruction.
- Review licensing at intake, before adding/updating external material, and again for the final artifact. Inventory direct/transitive runtime and development dependencies, copied/generated code, templates, fonts, icons, images, translations, sample data and delivered documentation. A development-only generator may still copy licensed code into production.
- Record origin, exact version/commit, actual license text, copyright holders, modifications, intended use, whether distributed, and required actions. Public access, an absent license, or AI generation is not proof of permission or provenance. Include client-provided assets and external services where relevant.
- Interpret SPDX OR/AND/WITH expressions correctly and record any chosen license. Do not approve an expression merely because it mentions MIT. Package metadata, a lockfile or an SBOM is evidence to inspect, not legal approval or proof of every bundled file's license.
- Check notice/attribution, modification notices, source-code or source-offer obligations, network-use conditions, commercial permissions and redistribution restrictions against actual use. Standard permissive licenses also have conditions. Copyleft, custom/dual licensing and unclear origin require individual assessment, not blanket approval or rejection.
- Keep required LICENSE/COPYING/NOTICE and attribution files in the production artifact despite the development-documentation exclusion. Verify their content and relevant source obligations after scoping, minification and final ZIP creation; renaming or prefixing does not remove license duties.
- Record outcome, evidence, reviewer and unresolved decisions against the source commit, lockfile fingerprints and final artifact hash. New versions, license changes, use changes or artifact changes invalidate the affected previous conclusion. Preserve confidential entitlement evidence without publishing customer secrets.
- An unresolved relevant obligation blocks release approval. Reviewers must report it as blocking, not informational. Raise a specific owner decision or qualified legal review when evidence cannot establish compatibility; AI must not grant its own waiver or fabricate consent. An inventory tool's success is not a release approval or a guarantee of legal compliance.

## Composer and packaging

- Use Composer PSR-4 autoloading for `<MODULE_SRC>` and classmap only legacy PrestaShop entrypoints that require it.
- Put test namespaces and test-only packages in `autoload-dev` and `require-dev`.
- The production module package must contain a generated `vendor/` when it has Composer dependencies. Load `vendor/autoload.php` from the module bootstrap and fail clearly if a required production autoloader is missing; do not maintain a second fallback autoloader.
- New modules must not depend on `avalanchemedia/prestashop-module-configuration`, `avalanchemedia/prestashop-module-hooks`, or `avalanchemedia/prestashop-module-installer`. Implement only the required installer, hook, and configuration adapters locally for the selected PrestaShop track. Do not remove these packages from existing modules without an explicit migration task.
- Prefer small module-local composition over copied universal base classes. Consider a shared runtime package only after the same substantial, stable behavior is used by at least three modules and has a versioned API plus CI for every supported PrestaShop track.
- Scope/prefix every third-party production dependency bundled with a module into the module namespace unless the selected PrestaShop version explicitly provides and supports that library. Exclude Composer/build plugins and development-only packages from the scoped production artifact.
- Keep production installs authoritative and optimized, use `prepend-autoloader: false` where supported, and exclude development dependencies. Test the final scoped production artifact, not only the unscoped source tree.
- Build the installable module ZIP from an explicit allowlist of runtime, installation and upgrade files. Include required production dependencies, assets, templates, translations, metadata and mandatory license/copyright/NOTICE files. Do not package the whole repository and rely only on exclusions.
- Exclude development and AI documentation (including `docs/`, `AGENTS.md` and other agent instructions), tests, fixtures, examples not needed at runtime, Docker/local-environment definitions, CI/editor/Git metadata, build and audit tools, test reports, backups, archives, logs, uploaded data, credentials and private keys. Keep development material in the repository and deliver user documentation separately; never commit secrets or private working data. Apply the same runtime-only policy to bundled dependencies while preserving their required licenses and notices.
- Run PrestaShop Autoindex after adding or moving module directories and again in the final distribution staging tree after installing/scoping dependencies. Include generated protective `index.php` files as required by Security / Directory and private-file protection; do not hand-edit generated guards or overwrite functional `index.php` entrypoints.
- Inspect the contents of the actual output ZIP, including hidden files and bundled dependencies. Automated package validation must fail on missing required files or directory guards, missing required private-directory protection, or prohibited development/work files. Test the extracted production artifact; a clean source tree or a successful ZIP command alone does not prove the package is ready.

## Admin controllers and routes

- Every back-office action, including AJAX actions, must declare authorization through the mechanism supported by the selected PrestaShop track. Map read/create/update/delete operations to the matching permission; securing only the menu or parent controller is insufficient.
- Bind each admin route to the correct installed tab/ACL identifier (`_legacy_controller` and `_legacy_link` where the track requires them). Install a hidden tab when an action needs permissions but no visible menu item.
- Use `GET` only for safe reads. State-changing actions use `POST` or another supported mutation method, validate CSRF, constrain route identifiers (for example to digits), and redirect after successful form writes.
- Prefer framework response objects such as `JsonResponse`. A legacy controller may use one shared response helper; do not scatter `die(json_encode(...))` across actions.

## Forms and native PrestaShop UI

### Back-office forms

- Implement new or redesigned module configuration pages with a Symfony controller, a dedicated form type, and a configuration data provider/handler for the declared PrestaShop track. Keep persistence and business rules out of the controller and form type. A legacy module configuration entrypoint may redirect to the Symfony route. A narrow fix to an existing legacy page does not require an unrelated migration; document any compatibility exception.
- Extend the native admin Twig layout and apply the PrestaShop form theme (for example, `@PrestaShop/Admin/TwigTemplateForm/prestashop_ui_kit.html.twig` where supported). Render fields through Symfony form helpers; use native field types, including `SwitchType` for boolean switches and appropriate choice types for selections. Reuse native cards, spacing, labels, buttons, help and validation messages instead of hand-building a parallel form design.
- Put help on the field it explains through the form's help mechanism. Keep labels associated with controls, preserve submitted values on validation failure, and display errors next to their fields. Follow the authorization, CSRF and redirect rules in Admin controllers and routes.
- Use the core's supported JavaScript components and assets. Initialize each widget only once; do not call Select2 again on fields already initialized by the native UI. Verify that the selected widget theme exists in the loaded admin CSS. Where needed, select it through field attributes: PS8/PS9 UI kit pages may require an explicit `data-theme="bootstrap4"` and `data-width="100%"` rather than the older Select2 default. Verify the actual target version instead of assuming identical defaults across tracks.
- Inspect a native configuration page and a relevant modern reference such as `am_limitorders` before designing the form. Reuse platform behavior, not a reference module's private configuration framework or copied core templates.

### Multistore configuration forms

- Make each shop/group-scoped setting participate in the native multistore form mechanism. Where supported, return `MultistoreConfigurationType::class` from the form type's `getParent()`, bind each scoped field to its canonical key using `multistore_configuration_key`, and initialize `MultistoreConfigField`. See the official configuration-form documentation for [PrestaShop 8](https://devdocs.prestashop-project.org/8/development/multistore/configuration-forms/) and [PrestaShop 9](https://devdocs.prestashop-project.org/9/development/multistore/configuration-forms/).
- In shop/group context, show the native override checkbox and inherited/disabled field state. In all-shops context, retain the native per-field dropdown showing shops/groups with explicit overrides; in narrower contexts retain the native inheritance information. Do not replace this with only a page-wide scope selector or a custom visual imitation.
- Load and save through the typed configuration service using the authorized current context. For settings using native PrestaShop inheritance, resolve shop override, then group override, then global value. Use native configuration storage and supported scope-aware APIs so the native form metadata and persisted overrides describe the same state; do not maintain a second independent override registry for the UI.
- A checked override checkbox saves an explicit value in the current scope. An unchecked override checkbox removes only that scope's override and restores inheritance. Keep the override flag separate from a boolean setting's value: `false`, `0`, an empty string or an empty list can be valid explicit values and must not mean "inherit". Changing a parent value must leave existing child overrides intact.
- Choose submission semantics deliberately. Clearing every selected option or unchecking a setting must persist the intended empty/false value. A complete settings form may need explicit `POST` handling because partial `PATCH` semantics can retain omitted fields. Distinguish intentionally cleared fields from inherited/disabled or unrelated fields; do not erase values merely because a partial request omitted them.
- With multistore disabled, present ordinary editable fields in the correct single-store configuration scope. During upgrades or explicit scoped operations, preserve existing group/shop overrides even if multistore is currently disabled; verify the chosen write API does not collapse them into global values. Migrations must retain explicit empty and false overrides as well as non-empty values.

### Front-office forms and account pages

- Use the active storefront theme's layout and form rendering conventions. On Smarty-based pages, prefer `FormField` data and the theme's `_partials/form-fields.tpl` renderer where available. Reuse the theme's markup for inputs, textareas, styled checkboxes, labels, buttons, help, errors and success notices. Do not bring the admin UI kit or a second Bootstrap/form framework into the storefront.
- Build account-page breadcrumbs through the controller's native breadcrumb API: preserve parent items, include the customer-account link, and append the translated current page title. Use the theme's normal account navigation and return links; do not hardcode URLs or draw a separate breadcrumb in the module template.
- Place field-specific help directly with that field using the theme's supported mechanism. Preserve entered values after validation errors, show errors in the native form layout, and keep labels, focus and keyboard interaction usable. Account settings require customer authentication and ownership checks; state-changing forms require CSRF protection under Security.
- Editable textareas and other fields must look and behave like the theme's enabled controls. Do not use disabled/greyed styling for an editable field or render a bare checkbox when the theme provides a styled checkbox. Follow the active theme's visual states rather than forcing a universal color scheme.
- If a supported theme/version lacks a required field renderer, extend only the smallest relevant template block and keep native wrappers and validation markup. Add narrowly scoped module CSS only when needed. Do not copy whole core/theme templates, edit the theme or core, or assume a field supported by one track exists in another.
- Write customer-facing labels and help around the customer's action. Explain account sharing or multistore scope only when it affects a decision the customer needs to make; do not expose internal shop-context details as boilerplate.

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

### Directory and private-file protection

- Protect the module root and every module subdirectory with an `index.php`, including asset directories, private working directories and bundled production dependencies. An existing functional `index.php` may remain and must be reviewed as an entrypoint; a generator must not overwrite it. For other directories use the standard generated protective file. An `index.php` is not access control for a file requested by its exact URL.
- Apply the same protection to directories created during installation, upgrades or normal operation, including nested directories. Create required guards before writing data, check creation failures, and do not leave private data in a publicly readable location after partial failure.
- Identify which files must be public and which are private. Prefer storing private uploads, logs, temporary exports, certificates/private keys and other sensitive working data outside the web document root. If a private directory is under a web-served path, ship or create an effective `.htaccess` at each required protection boundary that denies direct HTTP access to all files and descendants, directory listing and script execution. On Apache 2.4 use `Require all denied` and, where permitted, `Options -Indexes`; `Options -Indexes` alone does not prevent downloading a known filename.
- Ensure Apache actually honors the required `.htaccess` directives through `AllowOverride`/`AllowOverrideList`, and document equivalent access-denial configuration for other supported web servers. File presence alone is not proof of protection. Prevent child configuration from reopening a private subtree. Do not apply a blanket deny to public CSS, JavaScript, images or required module entrypoints.
- Serve private downloads only through an authenticated and authorized controller when the feature requires access. Check ownership and shop scope before reading the file; a hidden filename or knowledge of its URL is not authorization.
- Verify protection with harmless synthetic files in an isolated test environment: request a private directory, a known file, a nested file and a script fixture directly over HTTP without authentication. Access must be denied without returning private content, listing entries or executing the fixture; an HTTP 500 caused by invalid configuration is a failure, not protection. Also verify required public assets still load and authorized downloads still work. Cover shipped and dynamically created directories. Missing or unverified required protection blocks release.

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
