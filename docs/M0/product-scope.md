# Product scope and assumptions

**Status:** Proposed for review. Planning only; not an implementation specification until approved.

## 1. Product statement

TrazenFlow AI is a multi-tenant automation platform for designing, running, monitoring, and governing workflows that combine events, files, AI models, external services, and human decisions.

The product has two clients over one independently operated platform:

1. **Web SaaS:** the primary workspace for authoring workflows, viewing runs, managing files and integrations, responding to approvals, and administering workspaces.
2. **Windows desktop client:** an Electron application that provides the web experience plus carefully permissioned desktop capabilities such as notifications, a system tray, and user-authorized local-folder monitoring.

A browser-only wrapper is not an acceptable substitute for the Windows application. The desktop build must have an installer and tested native functionality.

## 2. Intended users and roles

Initial role model:

- **Workspace owner:** billing/plan administration, membership, security settings, and ownership transfer.
- **Workspace admin:** manage members, integrations, credentials, and workspace-level configuration.
- **Builder:** create and edit workflows; publish new workflow versions.
- **Operator:** run workflows and inspect executions within assigned permissions.
- **Approver:** approve or reject gated actions and review relevant context.
- **Viewer:** read-only access to explicitly permitted workspace resources.

Roles are starting concepts. Resource-level permissions and custom roles remain a design decision for later review.

## 3. Core functional scope

### 3.1 Workflow authoring

A visual editor based on a node-and-edge canvas. Planned node families:

- Event/manual/scheduled triggers
- AI model and agent steps
- File intake, extraction, and transformation
- Conditions and branching
- Bounded loops and iteration
- HTTP requests and webhooks
- Integration actions
- Human approval gates
- Wait/delay and workflow-end nodes

The editor must validate graph structure and node schemas before publication. Bounded loops are valid; uncontrolled cycles, missing exits, and unbounded recursion must be rejected or require explicit safe limits.

Published workflow versions are immutable. An execution is tied to the exact version and configuration snapshot it started with.

### 3.2 Execution engine

A durable asynchronous engine with separate worker processes. Requirements include:

- Durable job persistence
- At-least-once delivery with idempotent effects where possible
- Retries with bounded policy and jitter
- Worker lease/heartbeat and recovery after process loss
- Cancellation and timeout semantics
- Concurrency and rate limits
- Human approval pauses and resumption
- Execution event/history and structured logs
- Snapshotting of workflow version, inputs, and relevant execution configuration
- Tenant-isolated access to runs, logs, and outputs

Do not promise exactly-once execution across arbitrary third-party APIs. Use idempotency keys, deduplication, and explicit effect semantics; document actions that cannot be safely retried.

### 3.3 AI agents and model providers

An orchestration layer will support multiple providers through adapters, initially evaluating OpenAI, Anthropic, and compatible APIs. Provider choice must be configurable by the workspace or workflow policy.

Agents must have bounded tools, permissions, step counts, wall time, and model budgets. Require approval for sensitive or externally consequential operations. Model calls must record provider/model identifiers, usage when available, latency, outcome, and estimated cost; never log secret credentials or full sensitive prompts by default.

No model is assumed to be free. BYO-key and platform-managed billing are separate product decisions requiring explicit user approval.

### 3.4 Files and document intelligence

Planned capabilities:

- Resumable/multipart uploads
- File metadata, checksum, size and content-type verification
- Object storage separate from relational metadata
- Streaming and bounded-memory processing
- Extraction and chunking pipelines
- Search/index metadata with tenant isolation
- Retryable processing jobs and processing status
- Configurable file-size and workspace quota limits

Acceptance targets must distinguish a 100 MB functional test from a 1 GB configurable upper-limit and stress test. A 1 GB limit is not a promise that every provider, Replit workspace, storage tier, or free plan supports a 1 GB transfer.

### 3.5 Integrations

Planned integration families:

- REST API and outbound webhooks
- Gmail and Google Drive
- Notion
- Discord and Slack

OAuth scopes must be minimal and visible. Tokens must be encrypted and revocable. Webhook signatures, replay protection, rate limits, and tenant binding are required. Each integration must declare whether it supports retries and idempotency.

### 3.6 Windows desktop application

The Electron client is intended to provide:

- Windows installer and app lifecycle
- Secure preload/IPC boundary
- Native notifications and system tray
- User-authorized local folder selection and monitoring
- Clear pause/stop controls for local monitoring
- Safe file handoff to the remote platform
- Optional offline viewing of explicitly cached, non-sensitive UI state

The desktop client must not silently scan the user's disk, run hidden background collection, bypass OS permissions, or expose unrestricted Node.js access to renderer content.

## 4. Non-functional requirements

- Strong tenant isolation across database, storage, search, jobs, logs, and integrations.
- Explicit authorization on every server-side operation; frontend hiding is not authorization.
- Observable operations: structured logs, metrics, traces where feasible, and audit events.
- Recovery from worker/process interruption and transient provider failures.
- Secure file processing and protection against SSRF, path traversal, archive bombs, and hostile documents.
- Accessible, responsive web interface with a distinct, coherent Trazen design system.
- Documented data retention, export, deletion, and backup/recovery policy before production.
- Testable deployment separation for development, staging, and production.

## 5. Out of scope for the initial release

- Unbounded autonomous agents or unrestricted shell/tool execution
- Guaranteed exactly-once side effects against arbitrary integrations
- Unlimited free AI inference, storage, bandwidth, or workflow execution
- Mobile-native applications
- Arbitrary unattended local-machine control
- Marketplace features or TrazenMarket account/payment integration
- Production readiness based only on a successful preview build

## 6. Assumptions requiring confirmation

1. Initial target is a small-team SaaS, not an immediate high-volume enterprise platform.
2. Web and Windows clients share the same account, API, workflow, and execution model.
3. The Windows client is Electron, unless a later measured evaluation favours another framework.
4. PostgreSQL is the system of record; object storage holds file bytes.
5. A PostgreSQL-backed durable queue is preferred initially to avoid requiring Redis in the first release.
6. Replit is a candidate development environment, not a guaranteed free production host.
7. The existing GitHub repository `TreSaaS-DEV/TrazenFlow` is the intended source of truth.

Any assumption that proves false must be recorded and reviewed before it is silently changed.