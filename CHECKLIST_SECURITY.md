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
