# AI-assisted web app preflight checklist

This checklist is a triage aid, not a security guarantee or compliance certification. Record evidence for each answer; “the AI said so” is not evidence.

## 1. Critical user journeys

- [ ] List the three actions whose failure would most harm a user.
- [ ] Exercise each from a clean account and an expired session.
- [ ] Verify failures produce a safe, useful response rather than partial state.
- [ ] Add at least one automated happy-path and one failure-path test per action.

## 2. Authentication and authorisation

- [ ] Enforce access on the server, not only by hiding UI controls.
- [ ] Test that one account cannot read or modify another account's records.
- [ ] Expire and revoke sessions as documented.
- [ ] Keep secrets out of browser bundles, repositories, logs, and error messages.
- [ ] Rate-limit login, recovery, invitation, and other abuse-prone endpoints.

## 3. Data boundaries

- [ ] Validate request shape, size, and content server-side.
- [ ] Identify sensitive fields and where they are stored, logged, exported, and deleted.
- [ ] Make multi-step writes atomic or safely retryable.
- [ ] Test duplicate submissions, refreshes, timeouts, and concurrent requests.
- [ ] Restore a backup in a non-production environment; a backup is unproven until restored.

## 4. Dependencies and generated code

- [ ] Remove packages, routes, feature flags, and credentials left over from experiments.
- [ ] Review dependency install scripts and lock the dependency graph.
- [ ] Run the ecosystem's vulnerability and licence reports; triage rather than blindly suppress.
- [ ] Trace copied/generated authentication, payment, upload, and webhook code line by line.
- [ ] Search for TODOs, mocked success paths, placeholder IDs, and swallowed exceptions.

## 5. Operations

- [ ] Build and deploy from a clean checkout using documented commands.
- [ ] Pin runtime versions and capture required environment variable names without values.
- [ ] Add health checks that reflect required dependencies.
- [ ] Use structured logs with request correlation and redact secrets/user content.
- [ ] Define an owner and alert for failed critical journeys.
- [ ] Test rollback to the prior release.

## 6. User-facing quality

- [ ] Test keyboard-only navigation and visible focus on critical journeys.
- [ ] Check narrow mobile widths and slow/error states.
- [ ] Give forms persistent labels and useful validation messages.
- [ ] Measure the production build, not only the development server.
- [ ] Explain irreversible actions before they happen and require explicit intent.

## 7. Launch decision

For each unresolved item, write: evidence, affected users, mitigation, owner, and review date. Block launch only on concrete risk; do not turn an unranked scanner output into a release process.
