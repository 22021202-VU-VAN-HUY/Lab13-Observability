# Day 13 Observability Lab Report

> **Instruction**: All machine-readable tags are preserved. This submission was completed individually.

## 1. Team Metadata
- [GROUP_NAME]: Solo Observability Lab
- [REPO_URL]: https://github.com/22021202-VU-VAN-HUY/Lab13-Observability
- [MEMBERS]:
  - Member A: Vu Van Huy | Role: All roles (individual project)
  - Member B: N/A | Role: Individual project
  - Member C: N/A | Role: Individual project
  - Member D: N/A | Role: Individual project
  - Member E: N/A | Role: Individual project

---

## 2. Group Performance (Auto-Verified)
- [VALIDATE_LOGS_FINAL_SCORE]: 100/100
- [TOTAL_TRACES_COUNT]: 14 traces verified in a 120-minute window on June 15, 2026
- [PII_LEAKS_FOUND]: 0

---

## 3. Technical Evidence (Group)

### 3.1 Logging & Tracing
- [EVIDENCE_CORRELATION_ID_SCREENSHOT]: `docs/evidence/log-evidence.png`
- [EVIDENCE_PII_REDACTION_SCREENSHOT]: `docs/evidence/log-evidence.png`
- [EVIDENCE_TRACE_WATERFALL_SCREENSHOT]: `docs/evidence/langfuse-evidence.png`
- [TRACE_WATERFALL_EXPLANATION]: The `agent_run` root span has two children: `rag_retrieval` and `llm_generation`. During `rag_slow`, retrieval reached about 3.5 seconds while generation stayed near 150 ms, proving retrieval was the bottleneck. Raw prompt input/output capture is disabled; only scrubbed previews and usage metadata are sent.

### 3.2 Dashboard & SLOs
- [DASHBOARD_6_PANELS_SCREENSHOT]: `docs/evidence/dashboard.png`
- [SLO_TABLE]:
| SLI | Target | Window | Current Value |
|---|---:|---|---:|
| Latency P95 | < 3000ms | 28d | 154ms baseline; 3651ms during `rag_slow` |
| Error Rate | < 2% | 28d | 0% baseline; 33.33% during incident sequence |
| Cost Budget | < $2.5/day | 1d | $0.022056 for the 10-request baseline |
| Quality Score | >= 0.75 | 1d | 0.87 baseline |

### 3.3 Alerts & Runbook
- [ALERT_RULES_SCREENSHOT]: `docs/evidence/runbooks.png`
- [SAMPLE_RUNBOOK_LINK]: `docs/alerts.md#1-high-latency-p95`

---

## 4. Incident Response (Group)
- [SCENARIO_NAME]: `rag_slow`, `tool_fail`, and `cost_spike`
- [SYMPTOMS_OBSERVED]: `rag_slow` raised P95 to 3651ms; `tool_fail` returned HTTP 500 and produced `RuntimeError`; `cost_spike` raised maximum request cost to $0.009621.
- [ROOT_CAUSE_PROVED_BY]: Langfuse trace `46f6009a787411db6b09935af78ecb65` shows separate RAG and LLM observations. Structured logs identify `RuntimeError` for the tool failure. Token and cost metrics identify the output-token spike. Full values are in `docs/evidence/incident-results.json`.
- [FIX_ACTION]: Disable each incident toggle; add retrieval timeout/fallback, tool fallback/error handling, and output-token caps or cheaper model routing.
- [PREVENTIVE_MEASURE]: Alert on P95, error rate, and per-request cost; retain trace metadata, correlation IDs, PII scrubbing, and tested runbooks.

---

## 5. Individual Contributions & Evidence

### [MEMBER_A_NAME]: Vu Van Huy
- [TASKS_COMPLETED]: Windows setup, Python 3.14 dependency compatibility, correlation middleware, structured logging, recursive PII scrubbing, Langfuse v3 tracing, RAG/LLM waterfall, metrics, six-panel dashboard, SLOs, alerts, runbooks, incident automation, audit logs, tests, evidence, and report.
- [EVIDENCE_LINK]: [`46e1472` logging/PII](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/46e1472), [`b581a04` dashboard/SLO/alerts](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/b581a04), [`8cbfbe9` tracing/incidents](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/8cbfbe9), and [`bd1c698` tests/automation](https://github.com/22021202-VU-VAN-HUY/Lab13-Observability/commit/bd1c698).

### [MEMBER_B_NAME]: N/A
- [TASKS_COMPLETED]: Individual project; covered by Member A.
- [EVIDENCE_LINK]: N/A

### [MEMBER_C_NAME]: N/A
- [TASKS_COMPLETED]: Individual project; covered by Member A.
- [EVIDENCE_LINK]: N/A

### [MEMBER_D_NAME]: N/A
- [TASKS_COMPLETED]: Individual project; covered by Member A.
- [EVIDENCE_LINK]: N/A

### [MEMBER_E_NAME]: N/A
- [TASKS_COMPLETED]: Individual project; covered by Member A.
- [EVIDENCE_LINK]: N/A

---

## 6. Bonus Items (Optional)
- [BONUS_COST_OPTIMIZATION]: Cost spike is localized by output-token and max-cost metrics; mitigation is documented in the runbook.
- [BONUS_AUDIT_LOGS]: Successful chat actions and incident control actions are written separately to PII-scrubbed `data/audit.jsonl`.
- [BONUS_CUSTOM_METRIC]: `quality_avg`, per-request maximum cost, requests/minute, and an in-memory time series are exposed by `/metrics`. Incident and Langfuse verification are automated by scripts.
