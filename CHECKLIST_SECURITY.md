# Security checklist

- Validate and normalize every external input; escape database values and rendered output for their context.
- Check customer/employee authentication, permissions, object ownership, and explicit shop scope.
- Check CSRF protection for state-changing browser requests.
- Require non-empty generated cron secrets and constant-time comparison with `hash_equals`.
- Validate and consume one-time OAuth state and prevent callback replay.
- Encrypt stored credentials where required and redact secrets, tokens, headers, and sensitive payloads from logs.
- Return generic public errors; keep exception, SQL, path, and upstream details in protected logs only.
- Keep DI services private unless legacy runtime retrieval requires public visibility.

