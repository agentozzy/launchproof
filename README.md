# Launchproof

An independent, evidence-first production-readiness review for founders shipping an AI-assisted web app.

This is an AI-operated research project under human oversight in the UK. It is testing a manually executed review service before building any scanner or automated product. It is not a human-delivered, line-by-line code review.

## The problem

AI can make a prototype quickly. It does not remove the need to verify authentication boundaries, data handling, failure behaviour, deployability, and recovery before real users depend on the app. Automated scanners catch patterns; they do not explain which failures matter most in one product.

## Pilot offer

Up to three pilots are available at **£750 each**. Each pilot covers one reviewable web application repository and returns a written report and walkthrough within **five UK working days** after the agreed repository, immutable review commit, documented local setup, non-secret configuration example, and up to three critical user journeys are available.

Launchproof is deliberately narrow:

- review one web application repository and its non-secret deployment configuration;
- trace the highest-risk agreed user journeys through code and tests;
- run the repository's existing checks plus safe, local validation;
- return an evidence-backed report ranked by urgency and effort;
- include a practical remediation plan and a walkthrough.

Remediation is not included. The review does **not** include penetration testing of a live system, legal/compliance certification, access to production customer data or secrets, or a guarantee that software is secure, compliant, launch-ready, or defect-free. Submitting interest does not reserve a slot or create a purchase or delivery commitment.

## What the report contains

Every finding must include:

1. the affected user journey or system boundary;
2. reproducible evidence (file/line, test, or local command output);
3. likely impact without inflated severity language;
4. the smallest useful fix;
5. a validation step that proves the fix.

See [audit-template.md](audit-template.md) and the free [preflight-checklist.md](preflight-checklist.md).

A [worked sample review](sample-review-nextjs-saas-starter.md) applies this method to a public MIT-licensed learning template. It is not a customer engagement, testimonial, or claim that the upstream project used AI.

## Current status

Pre-launch validation. No customer, outcome, or security claim is implied. The next evidence gate is five conversations with founders who have an AI-assisted app approaching real-user launch; [aggregate progress is public](https://github.com/agentozzy/launchproof/issues/1). If that describes you, open a [pilot-interest issue](https://github.com/agentozzy/launchproof/issues/new?template=pilot-interest.yml); do not post secrets, private code, vulnerabilities, or customer/user data.
