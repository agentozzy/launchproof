# Sample production-readiness review: Next.js SaaS Starter

- **Repository:** <https://github.com/nextjs/saas-starter>
- **Review commit:** [`6e33e58`](https://github.com/nextjs/saas-starter/tree/6e33e58b1e553a41fe22e6b941a7229a002de361)
- **Review date:** 2026-08-08
- **Scope:** source, repository checks, and documented setup only
- **Why this repository:** it is a public MIT-licensed learning template with authentication, teams, Stripe, and deployment instructions. This is a demonstration review, not a customer engagement or a claim that the project used AI.

## Executive summary

The reviewed commit type-checks and has several good baseline controls. Before using it as the foundation of a real service, I would fix two user-journey failures: invalid invitation signup can leave an unusable account behind, and the invitation action reports success even though it does not deliver an invitation. I would also add a repeatable release gate around authentication, team, and payment paths.

This report deliberately avoids a generic score. Priorities describe release decisions for an application derived from the template, not criticism of a repository that explicitly calls itself a minimal learning resource.

## Validation log

| Command or observation | Result | Evidence |
|---|---|---|
| Clean shallow clone | Pass | Commit `6e33e58b1e553a41fe22e6b941a7229a002de361` |
| `pnpm install --frozen-lockfile --ignore-scripts` | Pass | 196 packages installed; pnpm reported its lockfile policy check passed for 289 entries |
| `pnpm exec tsc --noEmit` | Pass | Exit 0, no diagnostics |
| `pnpm run build` without configuration | Expected stop | Compilation and TypeScript passed, then page-data collection stopped because `POSTGRES_URL` was absent; the README documents required environment variables |
| Test files under the repository, excluding dependencies | None found | `find` over `*.test.*` and `*.spec.*` returned 0 |
| GitHub Actions workflow files | None found | `.github/workflows` absent at the reviewed commit |
| `test`, `lint`, or `check` package scripts | None defined | [`package.json`](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/package.json) |

## Findings

### LP-001 — Invalid invitation signup can consume the email before validation

- **Priority:** Before launch
- **Boundary:** invited-member signup
- **Evidence:** signup inserts the user before validating the invitation at [`actions.ts` lines 134–179](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/%28login%29/actions.ts#L134-L179). An invalid invitation returns an error after the insert. A later attempt sees the existing email and returns the generic creation failure at [lines 112–123](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/%28login%29/actions.ts#L112-L123).
- **Observed behaviour:** code-path review shows no transaction or compensating delete around the user insert and subsequent invitation check. This was not runtime-reproduced against a database.
- **Likely impact:** a mistyped, stale, or concurrently consumed invitation can leave the address registered without team membership, preventing a clean retry.
- **Recommended change:** validate and lock the invitation first, then create the user, accept the invitation, add membership, log activity, and establish the session inside one database transaction. Handle the unique-email race explicitly.
- **Proof of fix:** an integration test submits an invalid invite and proves no user was created; a second test runs two concurrent accepts and proves exactly one complete membership is committed.

### LP-002 — Team invitation reports success without delivering an invitation

- **Priority:** Before launch if invitations are enabled
- **Boundary:** owner invites a team member
- **Evidence:** the action inserts an invitation, contains a TODO for email delivery, then returns `Invitation sent successfully` at [`actions.ts` lines 439–457](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/%28login%29/actions.ts#L439-L457).
- **Observed behaviour:** the user-facing result says “sent” while the only delivery code is commented out.
- **Likely impact:** an owner believes onboarding progressed, while the intended member receives no link and cannot discover the numeric invitation ID through the documented journey.
- **Recommended change:** either implement delivery with observable success/failure, or relabel the feature as “create invitation” and provide a safe copy-link step. Do not return “sent” until the delivery provider accepts the message.
- **Proof of fix:** a journey test creates an invitation, captures the delivery adapter call, follows its URL, and verifies membership; a forced delivery failure must not display “sent”.

### LP-003 — Critical journeys have no repeatable release gate

- **Priority:** Before launch
- **Boundary:** authentication, teams, subscriptions, deployment
- **Evidence:** no repository test files or GitHub Actions workflows were present, and [`package.json`](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/package.json) defines no `test`, `lint`, or `check` script. Type-checking succeeds when invoked manually.
- **Observed behaviour:** a clean checkout has a build script but no committed automated proof for failure paths or cross-user/team boundaries.
- **Likely impact:** changes can silently break signup, invitation acceptance, team isolation, checkout completion, or webhook handling.
- **Recommended change:** add one documented `check` command and CI job. Start with database-backed integration tests for signup rollback, invitation races, cross-team member removal, checkout replay, and subscription webhook retries.
- **Proof of fix:** the same check runs from a clean checkout in CI and includes positive controls that fail when each boundary is deliberately weakened.

### LP-004 — Missing Stripe catalog data falls through to a broken purchase action

- **Priority:** Planned improvement; before launch if catalog changes independently
- **Boundary:** pricing to checkout
- **Evidence:** the pricing page displays fallback names/prices when expected Stripe products are absent at [`pricing/page.tsx` lines 15–46](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/%28dashboard%29/pricing/page.tsx#L15-L46), but passes the resulting undefined `priceId` into the form at [lines 88–90](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/%28dashboard%29/pricing/page.tsx#L88-L90). Checkout sends that value to Stripe as the line-item price at [`stripe.ts` lines 27–33](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/lib/payments/stripe.ts#L27-L33).
- **Observed behaviour:** source review shows a presentational fallback without a corresponding valid checkout identifier. This state was not exercised against Stripe.
- **Likely impact:** users can see a plausible plan and click “Get Started”, then receive an internal provider error instead of an actionable unavailable state.
- **Recommended change:** validate catalog configuration server-side and disable purchase actions with a clear unavailable message when a product or price is missing. Avoid displaying invented fallback economics.
- **Proof of fix:** a test supplies an empty/misnamed catalog and proves no checkout request is made and the page explains the unavailable state.

### LP-005 — Production guidance omits recovery and operability evidence

- **Priority:** Planned improvement
- **Boundary:** deployment and incident recovery
- **Evidence:** the [production section](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/README.md#going-to-production) covers Stripe, Vercel, and environment variables, but the reviewed repository has no health check, backup-restore exercise, rollback procedure, or alert definition.
- **Observed behaviour:** these controls are absent from the repository and documentation reviewed; they may exist in a deployer's external platform configuration.
- **Likely impact:** a derived app can be deployed without a shared definition of healthy service or tested recovery path.
- **Recommended change:** add a deployment checklist naming required health dependencies, migration ordering, rollback constraints, structured error reporting, alert owner, and a dated backup-restore test.
- **Proof of fix:** a non-production exercise restores a backup and rolls back one release while recording time, data loss window, and unresolved gaps.

## Positive controls observed

- Session tokens constrain the JWT algorithm and cookies are `httpOnly`, `secure`, and `sameSite=lax` ([`session.ts`](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/lib/auth/session.ts#L25-L58)).
- Stripe webhook signatures are verified before event handling ([webhook route](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/api/stripe/webhook/route.ts#L7-L21)).
- Team-member removal scopes the delete by both membership ID and the acting user's team ID ([`actions.ts` lines 361–388](https://github.com/nextjs/saas-starter/blob/6e33e58b1e553a41fe22e6b941a7229a002de361/app/%28login%29/actions.ts#L361-L388)).
- The lockfile installed reproducibly with lifecycle scripts disabled, and TypeScript passed at the reviewed commit.

## Scope and limitations

No production system, deployed demo, database, Stripe account, secrets, or customer data was accessed. No live penetration testing was performed. Findings LP-001 and LP-004 are code-path observations that require runtime regression tests before being treated as reproduced defects. This sample does not certify security, compliance, availability, or correctness, and it does not imply endorsement by or affiliation with the upstream maintainers.
