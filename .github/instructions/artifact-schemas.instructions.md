---
description: "Use when reading or writing pipeline artifacts (requirements.md, exploration.md, api-exploration.md, test-plan.md, test-cases.md, testcases.json) to keep their internal structure consistent across all Skills."
applyTo: "artifacts/**/*.md, artifacts/**/testcases.json"
---

# Artifact Content Contracts

These are the required section/field contracts for each pipeline artifact. Skills must produce output matching these contracts so downstream Skills can parse it reliably. See [artifact-naming.instructions.md](./artifact-naming.instructions.md) for file/folder naming.

## requirements.md (Jira Story Analyzer)

Required sections, in order: Story Summary, Functional Requirements, Business Rules, Acceptance Criteria, URLs, Validation Messages, User Workflows, Test Data, Test Scenarios, Risks, Assumptions, Open Questions.

- Acceptance criteria must be individually identifiable (numbered or bulleted) so `test-plan.md` and `testcases.json` can reference them by index or short code.

## exploration.md (Playwright Browser Exploration)

Required sections, in order: Discovered URLs, Locators, Navigation Flow, Forms, Validation Messages, Product Information, Application Map.

- Locators section must list each element with: page/component, description, locator strategy (role/label, `data-testid`, `id`, other), and locator value.

## api-exploration.md (API Contract Analyzer)

Required sections, in order: Base URLs, Authentication Schemes, Endpoints, Request Schemas, Response Schemas, Error Patterns, Rate Limits, Sample Payloads.

- **Base URLs**: List all base URLs/environments discovered (dev, staging, prod).
- **Authentication Schemes**: Detail authentication methods (API key, OAuth 2.0, JWT, Basic Auth) with parameter locations (header, query, body).
- **Endpoints**: For each endpoint:
  - HTTP method and path (e.g., `POST /api/v1/users`)
  - Description and purpose
  - Required authentication/authorization
- **Request Schemas**: For each endpoint:
  - Path parameters with types and constraints
  - Query parameters with types, defaults, and constraints
  - Required headers
  - Request body schema with field types, validation rules, and examples
- **Response Schemas**: For each endpoint:
  - Success status codes (200, 201, 204, etc.) with response body structure
  - Response headers of interest
  - Field descriptions and data types
- **Error Patterns**: Common error response structures by status code (400, 401, 403, 404, 500) with field mappings
- **Rate Limits**: Request limits, rate limit headers, and retry-after behaviour
- **Sample Payloads**: Working request/response examples for key endpoints

## test-plan.md (Test Plan Generator)

Required sections, in order: Positive Scenarios, Negative Scenarios, Boundary Tests, Execution Order, Dependencies, Risk Areas, Success Criteria, Priority.

- Every scenario must have a stable **Scenario ID** in the form `SC-###` (e.g. `SC-014`), assigned once and never reused for a different scenario.
- Every scenario must cite the requirement or acceptance criterion it traces back to.

## test-cases.md (Test Case Documenter - human readable)

For each test case: **Test Case ID** (`TC-###`, matching `testcases.json`), Title, Preconditions, Steps (numbered, with expected result per step), Overall Expected Result, Priority, Linked Scenario ID, Linked Requirement.

## testcases.json (Test Case Documenter - machine readable)

Must validate against [testcases.schema.json](../schemas/testcases.schema.json). Key rules:

- `id` matches the corresponding entry in `test-cases.md`.
- `linkedScenario` matches a Scenario ID from `test-plan.md`.
- `linkedRequirement` references acceptance criteria from `requirements.md`.
- `automationStatus` starts as `planned` and is updated to `automated` only after the Test Script Generator Skill produces a passing spec, or `blocked`/`not-automated` when generation is intentionally skipped.
