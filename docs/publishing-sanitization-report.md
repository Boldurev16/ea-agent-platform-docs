# Sanitization report — public documentation export

Date: `2026-06-07`  
Source: local `ea-agent-platform` project (documentation export only).  
Target: public repository `ea-agent-platform-docs`.

## Purpose

Document what was included, excluded, and generalized before publication.  
**No application source code** was copied.

---

## Included (published)

| Category | Location | Notes |
|----------|----------|-------|
| Landing & navigation | `docs/index.md`, `docs/navigation/` | GitHub Pages entry |
| Topic facades | `overview/`, `architecture/`, `runtime/`, `memory/`, `retrieval/`, `ingestion/`, `operations/`, `legends/`, `audit/` | Links to generated source |
| Generated docs | `docs/generated/` (01–15, cheat sheet, legends) | Sanitized |
| Diagram legends | `docs/generated/legends/*.md` | Russian explanatory text |
| Audit | `docs/generated/15-documentation-audit-report.md`, `13-open-questions.md` | Structural gaps, not secrets |
| Jekyll | `docs/_config.yml` | Pages build |
| Structure overview | `project-structure/folder-structure.md` | High-level folders only |

---

## Excluded (not published)

| Category | Reason |
|----------|--------|
| All application source (`app/`, `orchestration/`, `llm/`, …) | Code repository — not public docs |
| `.env`, `.env.example`, `docker-compose.yml` secrets | Credentials / deployment specifics |
| `data/` (chunks, reports, jsonl jobs) | Operational / environment-specific data |
| `scripts/`, `tests/`, `infra/` | Implementation and CI |
| `prompts/*.py` (full prompt text) | Business + implementation detail |
| Legacy dated docs (`2026-04-*`, `2026-05-31_*` internal paths) | Internal hosts, model paths, `C:\Users\...` |
| `docs/evaluation-quality-report.json`, `role-expansion-eval-report.json` | Eval artifacts |
| `docs/compose-smoke-runbook.md` | Concrete dev host/port matrix |
| `docs/legacy-docs-catalog.md` | Links to non-published internal docs |
| `plans/`, master plan files | Internal planning |
| Milestone / phase-2 full internal proposals | Summarized at high level only in overview |

---

## Generalized (replacements applied)

Automated and manual sanitization on markdown files:

| Original pattern | Public replacement |
|------------------|-------------------|
| `http://127.0.0.1:8000`, `localhost:8000` | `<application-base-url>` |
| `127.0.0.1:*`, specific dev ports | `<application-host>:<port>` placeholders |
| `host.docker.internal` | `<container-host-gateway>` |
| `ea_password`, `ea_user`, `minioadmin` | `<postgres-password>`, `<postgres-user>`, `<object-storage-access-key>` |
| `Норникель Спутник` | corporate EA reference |
| `УКА` | corporate EA function |
| KB snapshot `10410` | `<environment-specific-count>` |
| Specific model IDs (`Qwen/...`, GGUF paths) | `<chat-model-id>`, `<local-model-artifact>` |
| `C:\Users\PC\...` paths | `<local-model-path>` |
| `.env.example` | environment configuration template |
| `ea_documents` collection name | `<vector-collection-name>` |
| Links to `../plans/`, internal milestone files | Removed or marked not published |

---

## Intentionally high-level (kept abstract)

- Module paths (`orchestration/agent_stream.py`) — educational architecture references only; **no source copied**.
- API endpoint names (`POST /chat/agent`) — public integration surface.
- Env var **names** in contracts doc (e.g. `SESSION_STORE`, `STORAGE_PROVIDER`) — without secret values or internal URLs.
- Postgres job statuses, tool names (`search_kb`) — operational concepts.
- Audit report code evidence — file paths as documentation pointers, not downloadable code.

---

## Removed from public narrative

- Full corporate architect system prompt text
- Internal relevance filter logic details for legacy panel (token lengths) — referenced only as “legacy panel behavior”
- Concrete smoke script invocations with internal URLs
- Docker compose service dependency matrix with internal hostnames
- AnythingLLM clone paths and internal repo references

---

## Residual risk review

| Item | Status |
|------|--------|
| Generated docs still mention Python module paths | Accepted — architecture education |
| Deep dive references line numbers | Accepted — no source in repo |
| `14-deep-dive-priority-areas.md` is implementation-heavy | Included but sanitized; consider future public summary-only version |
| Mermaid diagrams may show port labels in unsanitized copies | Reviewed in `03-system-architecture` — use generic service names in public export |

---

## Re-export procedure

1. Update docs in private `ea-agent-platform` repo.
2. Re-run export script (see internal runbook) or copy `docs/` facade + `generated/` with sanitization rules above.
3. Run security grep: `password`, `api_key`, `secret`, `C:\Users`, private IPs.
4. Update this report.
5. Commit and push `ea-agent-platform-docs`.

[← Publishing guide](publishing-guide.md) · [← Home](index.md)
