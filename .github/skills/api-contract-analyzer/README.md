# API Contract Analyzer

## Purpose

Analyse Swagger/OpenAPI specifications, REST/GraphQL/SOAP endpoints, or API documentation and convert them into the structured `api-exploration.md` artifact that downstream API Skills depend on. This is the API equivalent of Playwright Browser Exploration.

## Inputs

- Swagger/OpenAPI specification (URL or file path)
- `requirements.md` (optional - for context)
- REST API endpoint documentation
- GraphQL schema
- SOAP WSDL

## Outputs

- `api-exploration.md`

## Artifacts Produced

- `api-exploration.md` - structure defined in [artifact-schemas.instructions.md](../../instructions/artifact-schemas.instructions.md). Contains discovered endpoints, request/response schemas, authentication requirements, headers, status codes, and sample payloads.

## Artifacts Consumed

- `requirements.md` (optional) - when present, use it to prioritise and validate which endpoints are relevant to the current user story.

## Execution Steps

1. Check whether `api-exploration.md` already exists in the request's `artifacts/<slug>/` directory and is structurally valid.
2. If valid and the API contract source hasn't changed, skip execution and reuse the existing file.
3. Otherwise, retrieve the API contract from the provided source:
   - For Swagger/OpenAPI: parse specification file locally or fetch from URL
   - For REST APIs without OpenAPI: analyse documentation or make exploratory requests to discover schema
   - For GraphQL: introspect schema via GraphQL introspection query
   - For SOAP: parse WSDL for operations, messages, and bindings
4. Extract API contract elements:
   - Base URL(s) and environment configuration
   - Authentication/authorization schemes (API keys, OAuth, JWT, Basic Auth)
   - Available endpoints/operations with HTTP methods
   - Request schemas (path parameters, query parameters, headers, body)
   - Response schemas (status codes, headers, body structure)
   - Error response patterns
   - Rate limits and constraints
   - Sample request/response payloads
5. If `requirements.md` exists, cross-reference to identify which endpoints are needed for the story's acceptance criteria.
6. Write `api-exploration.md`.

## Failure Handling

- Swagger/OpenAPI file unreachable or malformed: stop the pipeline and report the failure; do not fabricate a partial `api-exploration.md`.
- Authentication required to access API spec: document the authentication requirement and stop; do not proceed with incomplete data.
- Partial schema available: proceed but mark missing sections explicitly under Assumptions and Open Questions rather than guessing.

## Retry Strategy

- Retry transient network failures (timeouts, 5xx) up to 3 times with exponential backoff.
- Do not retry authentication failures or 4xx errors other than 429 (rate limit); surface them immediately.

## Logging

- Log: source type (OpenAPI/GraphQL/SOAP), number of endpoints discovered, authentication schemes identified, whether requirements.md was used for filtering, and whether the artifact was newly generated or reused.

## Future Extensions

- Support for AsyncAPI specifications (WebSocket, Kafka, AMQP).
- Live API contract testing via sample requests to validate documented behaviour.
- Contract drift detection between specification and live API responses.
- Multi-version API support (e.g., v1 vs v2 endpoints).
