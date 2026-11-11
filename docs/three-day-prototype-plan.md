# Three-day prototype delivery plan

**Status:** Draft planning proposal only. This document does not approve implementation, provisioning, external service activation, paid usage, or deployment.

## Reality check

The full TrazenFlow scope in the Milestone 0 product and architecture documents is not a credible three-day production build. It includes a multi-tenant workflow platform, durable asynchronous execution, AI provider governance, private file processing, integrations, and a genuine Windows desktop application.

A three-day target can only be treated as a **narrow web prototype/demo**. It must not be represented as production-ready, enterprise-ready, secure for real customer data, or feature-complete.

## Proposed three-day demo boundary

### Included, subject to a separate implementation approval

1. A web-only application; no Windows/Electron installer in this sprint.
2. One authenticated demo workspace, only if a separate TrazenFlow identity/database project is available and the required RLS checks can be completed.
3. A basic workflow canvas with a deliberately limited node set: Manual Trigger, one AI Text step, and Output.
4. Save and load a workflow draft.
5. Run the simple workflow manually and show step-by-step status/output.
6. One AI provider adapter, only if the user has an already-available key and the provider can be used without payment, card activation, or paid API usage. Never commit the key. Include a clear unavailable/configuration state if not.
7. Basic automated tests, a reproducible setup guide, and a demo script.
8. A deployable preview only if an already-available free service can be used without a card, paid plan, or auto-billing. Otherwise demonstrate locally or in the existing authorized Workik sandbox.

### Explicitly excluded from the three-day demo

- Production readiness or public customer onboarding.
- Windows/Electron application, installer, tray, notifications, or local folder monitoring.
- Arbitrary integrations (Gmail, Drive, Notion, Slack, Discord, etc.).
- File uploads, large-file handling, document extraction, embeddings, and object storage.
- Scheduled triggers, branching, loops, arbitrary HTTP actions, and unrestricted tool execution.
- A production-grade durable worker/queue with lease recovery and delivery guarantees.
- Billing, subscriptions, marketplace features, and every TrazenMarket service, credential, database, domain, or user record.
- Claims of complete multi-tenant security unless workspace isolation, RLS, and negative cross-workspace tests are actually implemented and pass.

## Day-by-day plan

### Day 1 — Establish a safe foundation

- Confirm the dedicated TrazenFlow repository is connected to Workik; do not connect TrazenMarket.
- Workik inspects the repository and reports the exact runtime/toolchain, existing files, and blockers before coding.
- After implementation approval, establish the smallest web application structure and environment-variable handling.
- Implement the agreed demo authentication/workspace boundary only if a dedicated TrazenFlow backend is provisioned and RLS tests are feasible.
- Create a working application shell and setup instructions.

**Day 1 gate:** application starts reproducibly; no secrets are committed; no TrazenMarket resources are referenced; tests and blockers are reported.

### Day 2 — Implement one complete vertical slice

- Implement the limited Manual Trigger → AI Text → Output workflow.
- Persist the draft and show its nodes/edges in the web interface.
- Execute only this bounded workflow path.
- Record run and per-step status with sanitized errors.
- Use one permitted AI provider only when a user-supplied, already-authorized key is available; otherwise clearly show provider-not-configured and keep a non-AI demo path separate.

**Day 2 gate:** a reviewer can create, save, reload, run, and inspect the demo workflow; failure states are visible and tests cover the main path.

### Day 3 — Stabilize and demonstrate

- Fix defects found in the PR review.
- Run type checks, lint, unit tests, API tests, and browser smoke tests that the environment supports.
- Verify authentication/authorization boundaries and add negative tests where applicable.
- Document limitations, setup, environment variables, and known issues.
- Produce a short demo walkthrough and a release candidate PR.
- Deploy only if an already-authorized free environment is available without payment or card activation.

**Day 3 gate:** evidence is attached to the PR; unresolved security/test failures are disclosed; the user decides whether to merge or demo.

## Workik execution and review protocol

1. Workik must receive tasks through the configured GitHub Issues integration or another explicitly verified Workik task trigger. Merely adding this document to GitHub does **not** submit or start a Workik task.
2. Workik works on a feature branch and opens a pull request. No direct commits to `main`.
3. Every task must state its scope, exact acceptance criteria, tests to run, and out-of-scope changes.
4. Do not merge PRs, deploy, provision services, enable paid features, or incur API charges automatically.
5. ChatGPT reviews the PR diff, related source files, test/CI evidence, and requirements. A code review is not a guarantee of zero defects.
6. The user makes the merge/release decision.

## Approval and stop rules

- This is a planning document, not Milestone 1 approval.
- Before implementation, the user must explicitly approve the narrowed three-day prototype scope and the corresponding implementation milestone.
- If a separate TrazenFlow backend, a free usable AI provider, or a no-card preview environment is unavailable, stop and report the blocker rather than borrowing TrazenMarket resources or silently substituting a paid service.
- Any scope expansion requires a new estimate and explicit approval.
- No secrets, private user data, or payment credentials may be placed in GitHub Issues or source files.
