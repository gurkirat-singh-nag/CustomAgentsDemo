# API Client Generator

## Purpose

Generate reusable API client classes or request-builder modules from `api-exploration.md` and `testcases.json`, creating the abstraction layer between test specs and raw HTTP/API calls. This is the API equivalent of Page Object Generator.

## Inputs

- `api-exploration.md`
- `testcases.json`

## Outputs

- `clients/`

## Artifacts Produced

- `clients/` - one client class per logical API resource or service, following the repository's API client conventions. Each client encapsulates:
  - Base URL and environment configuration
  - Authentication/authorization setup
  - Request builders for each endpoint (GET, POST, PUT, DELETE, etc.)
  - Response parsing and validation
  - Error handling patterns
  - Reusable headers, query parameters, and payload templates

## Artifacts Consumed

- `api-exploration.md` - source of truth for endpoints, schemas, and authentication.
- `testcases.json` - determines which endpoints need client methods; only generate clients for endpoints referenced in test cases, not the entire API surface.

## Execution Steps

1. Read `api-exploration.md` and `testcases.json`.
2. Group endpoints by logical resource or service (e.g., `UserClient`, `OrderClient`, `AuthClient`).
3. For each client class:
   - Check whether it already exists in `clients/`
   - If it exists and covers all required endpoints, skip regeneration
   - Otherwise, generate or update the client class with:
     - Constructor accepting base URL, authentication credentials, and configuration
     - One method per endpoint operation following naming conventions (e.g., `createUser()`, `getUserById()`, `updateOrder()`)
     - Type-safe request/response models (TypeScript interfaces, Java POJOs, Python dataclasses, etc.)
     - Built-in retry logic for transient failures
     - Structured error responses matching the API's error contract
4. Update `testcases.json` to populate the `requiredClients` field for each test case, listing which client classes and methods it depends on.
5. Write `clients/`.

## Failure Handling

- `api-exploration.md` missing or incomplete: stop the pipeline and report that API Contract Analyzer must run first.
- Endpoint schema ambiguous or malformed: flag specific endpoint as requiring manual review rather than generating incorrect code.
- Test case references an endpoint not present in `api-exploration.md`: stop and report the mismatch; do not fabricate the client method.

## Retry Strategy

- Pure code generation step; retry only transient file read/write errors (up to 3 attempts).

## Logging

- Log: number of client classes generated vs. reused, endpoints covered, and any schema ambiguities flagged for manual review.

## Code Generation Principles

- **Single Responsibility**: Each client class represents one logical API resource or bounded context.
- **Reusability**: Client methods are usable across multiple test cases; avoid test-specific logic in clients.
- **Type Safety**: Use strongly-typed request/response models wherever the language and framework support it.
- **Authentication Abstraction**: Authentication setup (tokens, API keys, OAuth flows) is handled in the client constructor or via a separate auth helper, not repeated in every request method.
- **Framework Agnostic**: Clients should use a common HTTP library (Axios, Fetch, RestSharp, Requests, etc.) and be usable outside the test framework if needed.

## Future Extensions

- GraphQL client generation with typed queries and mutations.
- gRPC client generation from proto files.
- WebSocket client generation for real-time APIs.
- Contract validation assertions built into client methods (e.g., response schema validation against OpenAPI spec).
