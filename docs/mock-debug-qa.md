# Demo and Debugging Q&A

## Why use correlation IDs?

The middleware clears request-local context, accepts a valid incoming `x-request-id` or generates `req-<8 hex>`, binds it to structlog, stores it on `request.state`, and returns it in the response header. This connects the client response, API logs, audit record, and incident investigation without storing user PII.

## How is P95 calculated?

The implementation sorts successful request latencies and applies the nearest-rank position. P95 represents tail latency: 95% of observed successful requests are at or below that value. It is more useful than an average for detecting a small number of very slow requests.

## How do metrics, traces, and logs work together?

Metrics show that an incident exists and quantify impact. Traces localize latency or failure to RAG versus LLM generation. Logs explain the request context and error type using the same correlation ID. The investigation order is Metrics -> Traces -> Logs.

## How is PII protected?

Regex patterns redact email, Vietnamese phone numbers, CCCD, credit cards, passports, IP addresses, and explicitly labeled addresses. Scrubbing is recursive across nested dictionaries and lists. User IDs are SHA-256 hashed. Langfuse decorators disable raw input/output capture and only attach sanitized previews and metadata.

## What caused each injected incident?

- `rag_slow`: retrieval sleeps for 3.5 seconds, so the RAG span dominates the trace.
- `tool_fail`: retrieval raises `RuntimeError("Vector store timeout")`, producing HTTP 500 and an error log.
- `cost_spike`: generated output tokens are multiplied by five, increasing per-request cost.

## Why is the audit log separate?

Operational logs support debugging, while audit logs record user-impacting actions and incident toggles. Separating them supports different retention, access control, and compliance policies.
