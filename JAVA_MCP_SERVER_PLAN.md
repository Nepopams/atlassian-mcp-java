# План: корпоративный Atlassian MCP Server на Java

## 1) Короткий ответ
Да, это реалистично: можно сделать Java-аналог официального `atlassian-mcp-server` и развивать его как отдельный корпоративный продукт.

---

## 2) Цели и границы

### Цели
- Функционально повторить ключевые сценарии официального сервера (Jira/Confluence/Compass через MCP).
- Добавить enterprise-возможности: управляемая аутентификация, аудит, наблюдаемость, SSO, ограничения по политикам.
- Обеспечить предсказуемые SLA для внутренних команд.

### Не-цели (первый релиз)
- Полная 1:1-совместимость со всеми edge-case и неофициальными фичами.
- Поддержка всех клиентов сразу — сначала 2–3 целевых клиента.

---

## 3) Что анализируем в Python-референсе

Базируемся на публичном репозитории Atlassian MCP Server:
- общий флоу подключения и transport (`/mcp`, legacy `/sse`),
- модель аутентификации (OAuth 2.1 + API token/headless),
- набор пользовательских сценариев (search/create/update Jira/Confluence/Compass),
- подход к security и permission boundaries.

На этом шаге делаем **матрицу совместимости**: feature-by-feature, где `Must / Should / Later`.

---

## 4) Целевая архитектура Java-сервера

### Технологический стек (рекомендуемый)
- Java 21 LTS
- Spring Boot 3.x
- WebFlux (SSE/streaming) или MVC + async (по нагрузке)
- OAuth2 Client + JWT validation (Spring Security)
- HTTP-клиент: Spring WebClient / OkHttp
- Конфиг: Spring Config + Vault/Secrets Manager
- Observability: Micrometer + OpenTelemetry + Prometheus/Grafana
- Логи и аудит: JSON logs + audit trail (DB/queue)

### Модули
1. **mcp-transport** — MCP endpoint, handshake, tool routing, streaming.
2. **auth-gateway** — OAuth/API token flows, session/token cache, refresh.
3. **atlassian-connectors** — Jira/Confluence/Compass adapters.
4. **tool-runtime** — registry tools, schemas, validation, retries/timeouts.
5. **policy-engine** — RBAC/ABAC, rate limits, tenant scoping.
6. **audit-observability** — аудит, метрики, tracing, алерты.

---

## 5) Roadmap (12 недель)

### Фаза 0 (Неделя 1): Discovery + Architecture Decision
- Разобрать Python-репозиторий и docs, зафиксировать обязательные сценарии.
- Подготовить ADR: transport, auth, deployment, data retention.
- Выход: дизайн-док + backlog эпиков.

### Фаза 1 (Недели 2–4): Core MCP + Auth
- Реализовать `/mcp` endpoint + базовый lifecycle.
- OAuth 2.1 и API token headless flow.
- Token/session management, secure secret handling.
- Выход: end-to-end demo с одним инструментом Jira search.

### Фаза 2 (Недели 5–7): Atlassian tools parity (MVP)
- Jira: search/create/update issue.
- Confluence: search/summarize/create page.
- Compass: базовые read/create операции.
- Выход: MVP с минимальной parity-матрицей.

### Фаза 3 (Недели 8–10): Enterprise hardening
- RBAC/policies, tenant isolation.
- Аудит действий (кто/что/когда), PII-safe logging.
- Rate limits, circuit breaker, retry strategy.
- Выход: pre-prod readiness checklist.

### Фаза 4 (Недели 11–12): Production launch
- Нагрузочное тестирование, chaos/failure drills.
- SLO/SLA, runbooks, on-call handoff.
- Поэтапный rollout (canary -> wide).
- Выход: GA v1.

---

## 6) Критерии готовности MVP
- MCP-клиент подключается и стабильно выполняет основные tools.
- p95 latency в целевом бюджете (например, <2с для read операций без больших payload).
- Ошибки аутентификации и авторизации корректно классифицируются и логируются.
- Полный audit trail для write-операций.
- Пройдены security checks (SAST/DAST/secret scan/dependency scan).

---

## 7) Риски и как закрываем
- **Изменения в официальном MCP/Atlassian API** → контрактные тесты + еженедельный compatibility scan.
- **Сложный OAuth/headless** → ранний PoC + отдельный auth test harness.
- **Перегрузка/429 со стороны Atlassian** → adaptive rate limiting + backoff.
- **Scope creep** → strict Must/Should/Later и feature freeze на MVP.

---

## 8) Команда
- 1 Tech Lead (Java + архитектура)
- 2 Backend Engineers
- 1 QA/SDET
- 0.5 DevOps/SRE
- 0.5 Security Engineer

Минимум на MVP: 4 FTE.

---

## 9) Практический старт на ближайшие 10 рабочих дней
1. Зафиксировать 15–20 ключевых use-case из официального сервера.
2. Поднять Java skeleton (Spring Boot, MCP endpoint, health/readiness).
3. Сделать Jira search tool end-to-end.
4. Подключить OAuth 2.1 + secure secrets.
5. Поднять CI: unit/integration/contract tests + quality gates.
6. Подготовить demo для архитектурного ревью.

---

## 10) Ответ на вопрос «можем ли?»
Да, можем. Рациональный путь: не «переписывать всё сразу», а идти через parity-матрицу и поэтапный MVP, сохраняя совместимость MCP и добавляя enterprise-требования с первого спринта.
