# Day 13 Observability Lab - Individual Submission

Completed individual implementation of the Monitoring, Logging, and Observability lab.

## What students will build

A small FastAPI "agent" instrumented with:
- structured JSON logging
- correlation ID propagation
- PII scrubbing
- Langfuse tracing
- minimal metrics aggregation
- SLOs, alerts, and a blueprint report

All starter TODOs are implemented. The final system includes automated verification, incident evidence, a six-panel dashboard, working runbooks, and a separate audit log.

## Completed lab scope

1. **Correlation IDs**: inbound propagation or `req-<8 hex>` generation, context binding, and response headers.
2. **Structured logs**: JSON schema, user hash, session, feature, model, environment, latency, tokens, cost, and error type.
3. **PII protection**: recursive redaction for email, phone, CCCD, card, passport, IP, and labeled address data.
4. **Langfuse tracing**: root agent span with RAG and LLM child observations; raw PII-bearing inputs are not captured.
5. **Metrics and dashboard**: six panels, 15-second refresh, SLO thresholds, time series, and alert badges.
6. **Incident response**: automated `rag_slow`, `tool_fail`, and `cost_spike` scenarios with runbooks.
7. **Submission evidence**: report, screenshots, machine-readable results, tests, and audit logs.

## Quick start

Requires Python 3.11 or newer. The dependencies include a prebuilt Windows wheel for Python 3.14, so Visual Studio Build Tools and Rust are not required.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
Copy-Item .env.example .env
python -m uvicorn app.main:app --reload --env-file .env
```

Add Langfuse project keys to `.env` before starting the server. Verify `http://127.0.0.1:8000/health` returns `"tracing_enabled": true`. Open the dashboard at `http://127.0.0.1:8000/` and API docs at `http://127.0.0.1:8000/docs`.

## Tooling

```powershell
# Generate requests (use --concurrency 5 to test parallel bottlenecks)
python scripts/load_test.py --concurrency 5

# Inject failures live
python scripts/inject_incident.py --scenario rag_slow

# Run all incidents and save evidence
python scripts/incident_demo.py --output docs/evidence/incident-results.json

# Check structured logs and PII
python scripts/validate_logs.py

# Verify Langfuse trace count and waterfall
python scripts/verify_langfuse.py --minutes 120 --minimum 10 --output docs/evidence/langfuse-summary.json

# Export readable JSON log evidence
python scripts/export_log_evidence.py

# Run tests and dependency validation
python -m pytest -q --basetemp=.pytest-tmp
python -m pip check
```

## Execution journal - June 15, 2026

### Part 1 - Environment and startup

Command: `python -m pip install -r requirements.txt`

Result: installation succeeded on CPython 3.14.4 after updating to `pydantic==2.13.4`; `pydantic-core==2.46.4` used a Windows wheel, so Rust/MSVC compilation was not required. `python -m pip check` reported `No broken requirements found`.

Command: `python -m uvicorn app.main:app --reload --env-file .env`

Result: `/health` returned `ok=true`, `tracing_enabled=true`, and all incident toggles were false.

### Part 2 - Normal traffic, logging, and PII

Command: `python scripts/load_test.py --concurrency 5`

Result: 10/10 requests returned HTTP 200 with 10 generated correlation IDs. Baseline metrics were P95 `154ms`, error rate `0%`, total cost `$0.022056`, and average quality `0.87`.

Command: `python scripts/validate_logs.py`

Result: 49 records analyzed, 0 missing required fields, 0 missing enrichment fields, 29 unique correlation IDs, 0 PII leaks, estimated score **100/100**.

### Part 3 - Langfuse tracing

Command: `python scripts/verify_langfuse.py --minutes 120 --minimum 10 --output docs/evidence/langfuse-summary.json`

Result: **14 traces**, passing the minimum of 10. The latest waterfall contains `agent_run` with child observations `rag_retrieval` and `llm_generation`.

### Part 4 - Incident response and alerts

Command: `python scripts/incident_demo.py --output docs/evidence/incident-results.json`

| Scenario | HTTP | Observed result | Alert |
|---|---:|---|---|
| `rag_slow` | 200 | P95 `3651ms` | `high_latency_p95` |
| `tool_fail` | 500 | Error rate reached `50%`, `RuntimeError` logged | `high_error_rate` |
| `cost_spike` | 200 | Max cost `$0.009621/request` | `cost_budget_spike` |

Result: all three alert rules fired as expected, each incident was disabled after the check, and final incident status returned all toggles false.

### Part 5 - Dashboard, tests, and evidence

Command: `python -m pytest -q --basetemp=.pytest-tmp`

Result: **7 passed**. Coverage includes correlation propagation, enrichment, recursive PII scrubbing, dashboard panel count, metric aggregation, alert firing, and tool failure accounting.

Evidence:

- Dashboard: `docs/evidence/dashboard.png`
- Langfuse count and waterfall: `docs/evidence/langfuse-evidence.png`
- Correlation and PII redaction: `docs/evidence/log-evidence.png`
- Alert runbooks: `docs/evidence/runbooks.png`
- Full report: `docs/blueprint-template.md`

## Repo map

```text
app/
  main.py                FastAPI app
  agent.py               core agent pipeline
  logging_config.py      structlog config
  middleware.py          correlation ID middleware
  pii.py                 scrubbing helpers
  tracing.py             Langfuse helpers
  schemas.py             request/response/log models
  metrics.py             in-memory metrics helpers
  alerts.py              live alert evaluation
  audit.py               separate PII-safe audit log
  dashboard.py           six-panel dashboard and runbook UI
  incidents.py           toggles for injected failures
  mock_llm.py            deterministic fake LLM
  mock_rag.py            deterministic fake retrieval
config/
  slo.yaml               final SLO definitions
  alert_rules.yaml       final alert rules
  logging_schema.json    expected log schema
scripts/
  load_test.py           generate requests
  inject_incident.py     flip incident toggles
  validate_logs.py       schema checks for logs
  incident_demo.py       automate all incident scenarios
  verify_langfuse.py     verify trace count and waterfall
  export_log_evidence.py export sanitized log samples
data/
  sample_queries.jsonl   requests for testing
  expected_answers.jsonl starter quality checks
  incidents.json         scenario descriptions
  logs.jsonl             app output target
  audit.jsonl            optional audit log output

docs/
  blueprint-template.md  team submission template
  alerts.md              runbook + alert worksheet
  dashboard-spec.md      6-panel dashboard checklist
  grading-evidence.md    evidence collection sheet
  mock-debug-qa.md       oral/written debugging questions
```

## Individual ownership

Vu Van Huy completed all roles: logging/PII, tracing, SLO/alerts, load testing, incident response, dashboard, evidence, report, and demo preparation.

## Git evidence

| Area | Commit |
|---|---|
| Structured logging, correlation IDs, PII, audit log | [`46e1472`](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/46e1472) |
| Dashboard, metrics, SLOs, alerts, runbooks | [`b581a04`](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/b581a04) |
| Langfuse waterfall and incident signals | [`8cbfbe9`](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/8cbfbe9) |
| Tests and verification automation | [`bd1c698`](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/bd1c698) |

## Grading policy (60/40 Split)

Your final grade is calculated as follows:

1. **Group Score (60%)**: 
   - **Technical Implementation (30 pts)**: Verified by `validate_logs.py` and live system state.
   - **Incident Response (10 pts)**: Accuracy of your root cause analysis in the report.
   - **Live Demo (20 pts)**: Team presentation and system demonstration.
2. **Individual Score (40%)**:
   - **Individual Report (20 pts)**: Quality of your specific contributions in `docs/blueprint-template.md`.
   - **Git Evidence (20 pts)**: Traceable work via commits and code ownership.

**Passing Criteria**: 
- [x] All starter TODO blocks completed.
- [x] `validate_logs.py` score is 100/100.
- [x] 14 traces verified in Langfuse.
- [x] Dashboard shows all 6 required panels.
- [x] Individual blueprint and evidence are complete.
