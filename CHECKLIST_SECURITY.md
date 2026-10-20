# Security checklist

- Validate and normalize every external input; escape database values and rendered output for their context.
- Check customer/employee authentication, operation-specific admin permissions, object ownership, explicit shop scope, and visible/hidden tab ACL mapping for every endpoint including AJAX.
- Check that state-changing routes reject `GET`, use supported mutation methods, and validate CSRF.
- Require non-empty generated cron secrets and constant-time comparison with `hash_equals`.
- Validate and consume one-time OAuth state and prevent callback replay.
- Encrypt stored credentials where required and redact secrets, tokens, headers, and sensitive payloads from logs.
- Return generic public errors; keep exception, SQL, path, and upstream details in protected logs only.
- Check outbound TLS verification, bounded retries, correlation IDs, and sanitization before API payloads enter exceptions or logs.
- Keep DI services private unless legacy runtime retrieval requires public visibility.
- Apply CONVENTIONS.md (Directory and private-file protection): verify `index.php` coverage for the module root, all subdirectories and bundled dependencies, including directories created during installation, upgrades and runtime. Preserve legitimate entrypoints and confirm safe handling of guard-creation failures.
- Inventory private working directories such as uploads, logs, temporary exports and certificate/key storage. Prefer storage outside the web document root; otherwise verify effective `.htaccess` access denial or equivalent server protection for the complete subtree. Do not treat a directory index guard, disabled listing or an unguessable filename as protection of individual files.
- Using harmless synthetic data in an isolated environment, verify anonymous HTTP requests cannot list a private directory, download known or nested files, or execute a script fixture. Verify the server honors the configured rules, including child directories; HTTP 500 is not a passing result. Check required public assets and authorized downloads remain functional.
- Inspect the final ZIP for secrets, private keys and private working data, and confirm required protection files survive packaging. Record evidence of the HTTP checks; missing or unverified required protection blocks release.
