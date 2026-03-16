<!--
SYNC IMPACT REPORT

Version change: TEMPLATE → 1.0.0
Modified principles:
- Project name: ContosoDashboard
- Added explicit training-first, security, spec-driven, offline-first, and simplicity principles
Added sections:
- Constraints (Training & Deployment)
- Development Workflow (spec-driven / PR requirements)
Templates updated:
- .specify/templates/plan-template.md ✅ updated
- .specify/templates/spec-template.md ✅ updated
- .specify/templates/tasks-template.md ✅ updated

-->

# ContosoDashboard Constitution

## Core Principles

### 1. Training-First Clarity (NON-NEGOTIABLE)
All code, documentation, and design decisions MUST be obvious to someone learning the platform.
- Implement with minimal assumptions; prefer explicit over clever.
- Document intent in code comments, README, and relevant spec artifacts.
- Avoid abstractions that add cognitive load unless they demonstrably reduce long-term complexity.

### 2. Security & Privacy by Design (NON-NEGOTIABLE)
Security is not optional; security controls MUST be built-in and verified continuously.
- Enforce authentication and authorization at the boundary (e.g., `[Authorize]` on pages, service-layer checks).
- Protect against common web risks (IDOR, XSS, CSRF, sensitive data exposure) in both UI and APIs.
- Treat mock implementations as training-only; highlight where production-grade substitutes are required.

### 3. Spec-Driven Development (SDD) & Testability (NON-NEGOTIABLE)
Every change MUST be guided by a written specification and supported by concrete, verifiable tests.
- Use `/speckit.specify` to create a feature spec with user stories, acceptance criteria, and success metrics.
- Ensure each user story is independently testable and has clear pass/fail criteria.
- Add or update automated tests when behavior changes; tests are the contract.

### 4. Offline-First with Cloud Migration Path
The project MUST run fully offline while remaining architected for future cloud adoption.
- Use abstractions (interfaces/services) for infrastructure dependencies (storage, auth, config).
- Local implementations (SQLite, file system, mock auth) are acceptable for training, but must be swappable.
- Document migration steps for each abstraction in the relevant spec or architecture docs.

### 5. Simplicity, Maintainability & Incremental Delivery
Prefer the simplest change that delivers value; avoid premature optimization.
- Break work into small, self-contained increments that can be delivered and rolled back safely.
- Refactor only when it improves clarity or enables future work; avoid speculative refactors.
- Keep dependencies minimal and explicit; remove unused code or packages promptly.

## Constraints & Deployment
- This repository is a **training artifact**; do not treat it as production-grade.
- No external service dependencies are allowed for core functionality; all critical paths must work offline.
- Production readiness (e.g., real identity providers, cloud storage) is explicitly out of scope unless documented as a future migration.
- Keep the tooling chain simple: rely on .NET SDK, VS Code, and built-in CLI tools.

## Development Workflow & Review Process
- Work begins with `/speckit.specify` to define a clear spec, followed by `/speckit.plan` to plan implementation.
- Feature branches MUST reference the spec and plan in PR descriptions.
- PRs MUST include:
  - Links to the related `/specs/*/spec.md` and `/specs/*/plan.md` documents.
  - A brief notes section explaining deviation from existing principles (if any).
  - Tests that verify new behavior and guard against regressions.
- Code reviews MUST verify alignment with this constitution; reviewers are responsible for calling out violations.

## Governance
- This Constitution is the authoritative guidance for how work is planned and implemented in this repository.
- Changes to this document require a PR with:
  1. A clear rationale for the change.
  2. An updated Sync Impact Report (as an HTML comment at the top of this file).
  3. Updates to any dependent templates or guidance files that are affected.
- Versioning:
  - MAJOR: Breaking changes to principles or governance.
  - MINOR: Added principles, sections, or materially expanded guidance.
  - PATCH: Language clarifications, typo fixes, or formatting improvements.

**Version**: 1.0.0 | **Ratified**: 2026-03-16 | **Last Amended**: 2026-03-16
