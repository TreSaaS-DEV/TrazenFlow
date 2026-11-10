# TrazenFlow AI

**Status:** Milestone 0 planning only — implementation is not approved.

TrazenFlow AI is planned as an AI automation platform with a web SaaS and a genuine Windows desktop client. This repository is the intended source of truth for planning and, after approval, implementation.

## Project controls

- Work proceeds one milestone at a time.
- Each milestone has explicit acceptance criteria and a review gate.
- A milestone is not approved merely because the preceding one was approved.
- No application implementation, dependency installation, infrastructure provisioning, paid service activation, or deployment is part of Milestone 0.
- TrazenFlow must use infrastructure and credentials separate from TrazenMarket. TrazenMarket production systems, databases, payment services, storage, secrets, domains, and user data are out of scope and must not be accessed or reused.

## Milestone 0 documents

- [Product scope and assumptions](docs/M0/product-scope.md)
- [Architecture and technical decisions](docs/M0/architecture.md)
- [Conceptual data model](docs/M0/data-model.md)
- [Security and threat model](docs/M0/security-threat-model.md)
- [Testing and acceptance criteria](docs/M0/testing-and-acceptance.md)
- [Cost and feasibility](docs/M0/cost-and-feasibility.md)
- [Milestone roadmap and approval gates](docs/M0/roadmap.md)

## Current execution status

- GitHub repository was verified as public, default branch `main`, with no commits or branches before this planning commit.
- Replit app creation through the connected Replit integration was attempted after approval but rejected by the service with `requires_active_subscription`. No Replit app was created.
- This documentation-only fallback does not imply that the Replit connection is working.
- No application code, dependencies, secrets, cloud resources, databases, storage buckets, or deployments have been created.

## Required next decision

Review the Milestone 0 documents and approve or request changes. After M0 approval, the next separate approval will be required before Milestone 1 implementation begins.
