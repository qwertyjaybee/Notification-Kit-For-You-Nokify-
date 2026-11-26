# Nokify project rules

These rules apply throughout the repository. Read [backend rules](api/AGENTS.md)
or [frontend rules](ui/AGENTS.md) before changing that application. Scoped rules
extend these conventions. Preserve framework-managed instruction blocks.

## Boundaries and readable code

- `api/` owns persistence, business operations, validation, and authorization;
  `ui/` owns presentation and user interaction.
- Documented folder structures are targets. Create folders only when real code
  needs them; do not scaffold empty layers or speculative abstractions.
- Keep changes focused. Avoid unrelated refactors, dependencies, and formatting churn.
- Give modules and functions clear responsibilities. Extract cohesive logic when
  responsibilities mix, not merely to satisfy arbitrary line limits.
- Prefer early returns, explicit braces, and readable multiline code. Avoid deep
  nesting, clever expressions, and dense one-line implementations.
- Use descriptive names: `notificationRecipient`, `sendNotification`, `isEnabled`.
  Avoid vague names such as `data` or `process` when a precise name is possible.
- Use `camelCase` functions, methods, parameters, and variables; `PascalCase`
  classes and types; `UPPER_SNAKE_CASE` constants. Boolean names start with `is`,
  `has`, `can`, or `should`. Follow scoped and framework conventions where required.
- Keep dependencies and side effects explicit. Avoid circular dependencies,
  catch-all utility files, duplicated business logic, and unnecessary service layers.
- Share code for actual reuse. Keep feature-specific logic with its feature.
- Comments explain reasons and constraints. Remove dead code; do not leave
  commented-out implementations or silently swallow errors. Never fake success.

## Security requirements

These requirements must be implemented and tested as features are introduced.
Their presence in documentation does not prove that the application is secure.

- Validate untrusted input on the server, including route parameters, uploads,
  webhooks, and external responses. Client validation is a usability aid.
- Authenticate protected operations and authorize every affected record, including
  reads. Hidden buttons are not access control. Never trust submitted ownership,
  roles, permissions, or tenant IDs to establish access.
- Write only explicitly allowed, validated fields. Parameterize queries; never
  interpolate untrusted input into SQL or shell commands.
- Keep secrets out of Git, browser bundles, URLs, logs, and error responses.
  Commit placeholder environment examples only. Rotate exposed credentials;
  deleting the current copy does not revoke them.
- Use framework password hashing and established cryptography, not custom schemes.
- Escape rendered output. Raw HTML or SQL needs explicit justification, safe input
  handling, and relevant tests.
- Choose and document authentication architecture before protected features.
  Cookie authentication requires framework CSRF protection and appropriately
  configured Secure, HttpOnly, and SameSite cookies.
- Restrict CORS to required origins, methods, and headers. Never use unrestricted
  origins with credentialed requests. CORS is not authorization.
- Rate-limit authentication and abuse-prone or expensive endpoints. Verify webhook
  signatures and prevent replay when the provider supports it.
- For uploads, restrict size and content types, generate storage names, prevent
  executable uploads, and authorize private downloads.
- Validate user-controlled outbound URLs and redirects; prevent external URL
  features from accessing internal services.
- Require production HTTPS, disabled debug output, least-privilege credentials,
  and redacted logs and error responses.
- Review dependencies and commit lockfiles. Assess audit findings; do not blindly
  apply breaking upgrades or claim an audit proves security.

## Completion checklist

- Check structure, naming, readable code, and the diff for secrets/unrelated files.
- Add meaningful tests for changed behavior. For protected features, cover success,
  invalid input, unauthenticated requests, forbidden actions, and attempts to access
  another user's records.
- Run relevant checks from the scoped rules. Report exactly what passed, failed,
  or could not run. Never present skipped checks as passed.
- Document new configuration and distinguish implemented features from plans.
- Documentation-only changes require link, consistency, and diff review; they do
  not require installing runtimes/packages or running application builds/tests.
