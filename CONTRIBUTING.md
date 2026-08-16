# Contributing

## Workflow

1. Keep `main` deployable.
2. Create a focused branch per feature/fix.
3. Link work to a GitHub issue where practical.
4. Keep commits small and descriptive.
5. Open a pull request with summary, testing, risks, and screenshots for UI changes.
6. Do not merge failing CI.

## Branch examples

```text
feat/auth-otp
feat/job-posting
feat/resume-cache
feat/ai-matching
feat/esewa-payments
fix/application-authorization
```

## Commit convention

Use concise conventional prefixes where useful:

```text
feat: add candidate application endpoint
fix: prevent duplicate payment entitlement
docs: document AI cache invariant
test: cover cross-company authorization
chore: configure CI
```

## Pull request checklist

- [ ] Requirements/acceptance criteria satisfied
- [ ] Authorization considered
- [ ] Input validation included
- [ ] Database migration included if schema changed
- [ ] Tests added/updated
- [ ] Error/failure paths handled
- [ ] No secrets or sensitive data committed/logged
- [ ] Documentation updated where architecture/API behavior changed
- [ ] UI tested on relevant mobile/desktop sizes

## Security

Never commit credentials. If a credential is accidentally committed, assume it is compromised and rotate/revoke it; deleting the file in a later commit is not sufficient.

## Database changes

All schema changes use migrations. Avoid destructive changes without a data migration and rollback/forward recovery plan.

## AI changes

Any change to matching behavior must version the prompt/matching contract where needed and preserve strict structured output validation. Do not add autonomous hiring/rejection behavior.

## Payment changes

Payment changes require explicit tests for replay/idempotency and server-side verification. Never grant entitlement from client-side success state alone.
