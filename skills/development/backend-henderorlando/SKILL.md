---
name: backend
description: Skill para diseñar y construir backends y APIs **modulares**, **auditables**, **resilientes** y **multi-tenant**, con seguridad fuerte, modelado correcto (incl. ledger cuando aplique), datos y jobs/colas operables.
---

# 🧱 Windsurf Skill — Backend & APIs (Columna vertebral)
**Skill ID:** SK-BE-API-001  
**Aplica a:** Fintech, Legaltech, Edtech, Healthtech, Retailtech, Proptech, Foodtech, Medtech, Regtech  
**Objetivo:** diseñar y construir backends y APIs **modulares**, **auditables**, **resilientes** y **multi-tenant**, con seguridad fuerte, modelado correcto (incl. ledger cuando aplique), datos y jobs/colas operables.

---

## 0) Backend Profile (output obligatorio)
Antes de diseñar/codificar, Windsurf debe fijar:

- **API style:** REST | GraphQL | híbrido
- **Versioning:** URL (`/v1`) | header | schema version (GraphQL)
- **Stack:** NestJS | FastAPI | Go | Elixir | otro (definir)
- **Architecture:** DDD + Hexagonal (default) | otra (justificar)
- **CQRS:** off | on (por módulos) + justificación
- **Eventing:** none | async events (outbox) | full EDA
- **Auth:** OAuth2/OIDC + provider (Keycloak/Auth0/Cognito/otro)
- **Multi-tenant:** single-DB shared | schema-per-tenant | DB-per-tenant (definir)
- **Data:** Postgres/MySQL/TiDB + Redis + object storage (si aplica)
- **Jobs/Queue:** BullMQ | SQS | Rabbit | Kafka | otro
- **Observability:** OTel + logs + metrics + tracing
- **Risk Tier:** R0–R3 (según regla base)

> Gate: sin Backend Profile explícito, Windsurf **no avanza**; declara supuestos “Hard” + impacto.

---

## 1) Principios (no negociables)
1. **API contract first:** schema + errores + paginación + auth antes de implementar.
2. **Dominio puro:** reglas core viven en `domain/`, sin frameworks.
3. **Writes son sagrados:** idempotencia, validaciones, auditoría y consistencia.
4. **Multi-tenant by design:** toda consulta y comando está “scoped” por `tenantId`.
5. **Operabilidad:** logs estructurados + tracing + métricas + health checks desde MVP.
6. **Evolución segura:** versionado de APIs, migraciones expand/contract, backward compatibility.

---

## 2) Diseño de APIs (REST/GraphQL)
### 2.1 Reglas generales
- **Recursos y verbos:** REST orientado a recursos; GraphQL para composición de lectura.
- **Versionado obligatorio:** no romper contratos sin versión nueva.
- **Consistencia de responses:**
  - `data`, `meta`, `errors` (estructura estándar)
- **Errores tipados:**
  - `DOMAIN_*`, `VALIDATION_*`, `AUTH_*`, `RATE_LIMIT_*`, `INTEGRATION_*`

### 2.2 Paginación y filtros
- Preferir **cursor pagination** para listas grandes; offset solo para admin pequeño.
- Filtros composables: `?status=&from=&to=&q=`.
- Sorting explícito (`sort=createdAt:desc`).

### 2.3 Idempotencia (obligatoria en writes críticos)
- Header `Idempotency-Key` (y/o `X-Request-Id`)
- Persistir resultado por key (Redis o DB) con TTL + estado
- Retornar la misma respuesta para reintentos

**Gate API (bloquea):**
- Endpoint transaccional sin idempotencia (cuando aplica).
- Listas sin paginación.
- Errores no estandarizados o sin códigos.

---

## 3) Arquitectura (DDD/Hexagonal + modularidad)
### 3.1 Capas canónicas
- `domain/`: entities, value objects, domain services, invariants, ports, domain events
- `application/`: use cases / handlers (CQRS), DTOs, policies, orchestration
- `infrastructure/`: adapters (db/http/queue), persistence, clients, config
- `interfaces/`: controllers/resolvers/consumers

### 3.2 CQRS (cuando aplica)
**Activar CQRS si:**
- Lecturas complejas (reporting, dashboards) difieren del modelo de escritura
- Necesitas escalado independiente read/write
- Event-driven y proyecciones

**Reglas CQRS:**
- Commands no retornan vistas pesadas (solo IDs o resumen)
- Queries no mutan estado
- Proyecciones versionadas + rebuild strategy

**Gate arquitectura (bloquea):**
- Dominio depende de ORM/framework.
- Controllers contienen reglas core.
- CQRS activado sin necesidad (complejidad sin ROI).

---

## 4) AuthN/AuthZ (OIDC, MFA, RBAC/ABAC)
### 4.1 Autenticación (AuthN)
- OAuth2/OIDC obligatorio (tokens firmados, introspection si aplica)
- MFA opcional: requerido por `R2+` o acciones sensibles
- Sessions/refresh tokens con rotación

### 4.2 Autorización (AuthZ)
**RBAC mínimo:**
- Roles por tenant (Owner/Admin/Member/Viewer, etc.)
- Permisos por acción (CRUD + export + billing + settings)

**ABAC (cuando aplique):**
- Reglas por atributo: `tenantId`, `plan`, `resourceOwnerId`, `region`, `featureFlags`
- Policy engine (simple) o middleware centralizado

**Gate Auth (bloquea):**
- Endpoints sin verificación de `tenantId` y permisos.
- Token handling inseguro.
- Acciones sensibles sin MFA (si definido).

---

## 5) Multi-tenant (pymes): aislamiento, roles, límites, planes
### 5.1 Estrategias de aislamiento (declarar una)
- **Shared DB + tenant_id** (default MVP)
- **Schema-per-tenant** (aislamiento medio)
- **DB-per-tenant** (aislamiento fuerte / enterprise)

### 5.2 Reglas duras multi-tenant
- Todo request debe derivar `tenantId` desde el token/contexto.
- Toda query y write deben filtrar por `tenantId` (sin excepciones).
- Auditoría incluye `tenantId`, `actorId`, `role`, `resource`, `outcome`.

### 5.3 Planes y límites
- Feature gates por plan (free/pro/enterprise)
- Rate limits por plan + cuotas (storage, exports, seats)
- Soft limits con warnings + hard limits con errores claros

**Gate multi-tenant (bloquea):**
- Cualquier endpoint accede datos sin scope de tenant.
- Falta de modelo de roles por tenant.

---

## 6) Modelado financiero / Ledger (cuando aplique)
> Recomendado para PyMEs y dominios donde la contabilidad sea core.

### 6.1 Doble partida (ideal)
- `Account` (chart of accounts)
- `JournalEntry` (cabecera)
- `Posting/Line` (debe balancear: sum(debits) = sum(credits))
- `Money` VO (amount + currency) con reglas de redondeo

### 6.2 Invariantes (obligatorias si hay ledger)
- Toda entrada debe estar balanceada.
- No mutar entradas posteadas: usar reversal/adjustments.
- Idempotencia por `externalRef` (imports, integraciones).
- Periodos de cierre (si aplica) + permisos.

### 6.3 Reconciliación
- Matching por `externalId`, fecha, monto, contrapartida
- Estados: `unreconciled`, `matched`, `manual_adjusted`
- Reportes reproducibles (misma query → mismo resultado)

**Gate ledger (bloquea si ledger activo):**
- No existe regla de balanceo.
- Se “edita” el pasado sin trazabilidad (sin reversal).
- No existe estrategia de reconciliación/import.

---

## 7) Bases de datos (Postgres/MySQL): índices, reporting, migraciones
### 7.1 Reglas de schema
- IDs estables (UUID/ULID) + timestamps + `tenantId`
- Constraints: FK, unique, check constraints (cuando aplica)
- Índices en:
  - (`tenantId`, `createdAt`)
  - campos de búsqueda/filtros
  - claves naturales (external refs)

### 7.2 Reporting queries
- Separar queries de reporting del path transaccional si pesan.
- Materialized views / read models (si CQRS).
- Evitar N+1; usar joins y prefetch controlado.

### 7.3 Migraciones
- **Expand/Contract** (compatibilidad hacia atrás)
- Migraciones idempotentes
- Tests de migración en CI (cuando posible)

**Gate DB (bloquea):**
- Tablas sin `tenantId` en sistemas multi-tenant.
- Migraciones sin strategy safe deploy.
- Índices ausentes en rutas de consulta críticas.

---

## 8) Jobs/Colas (recordatorios, cálculos, agregaciones)
### 8.1 Selección de tecnología (declarar una)
- BullMQ (Redis) para MVP rápido
- SQS/Rabbit para colas gestionadas / integración enterprise
- Kafka para EDA + streams y alto throughput

### 8.2 Reglas duras
- Jobs idempotentes (jobKey / dedupe)
- Retries con backoff + jitter
- Dead-letter queue (DLQ) o parking queue
- Observabilidad: métricas de lag, fail rate, processing time
- Rate limit / concurrency control

### 8.3 Tipos comunes
- Recordatorios/notifications
- Cálculos e insights (agregaciones periódicas)
- Sync/import de datos externos
- Rebuild de proyecciones (si CQRS)

**Gate jobs/queues (bloquea):**
- Jobs sin idempotencia o sin DLQ.
- Reintentos sin límite ni backoff.
- Sin métricas de cola/worker.

---

## 9) Event-driven (cuando se active)
### 9.1 Outbox pattern (mínimo)
- Insert event en outbox en la misma transacción del write
- Publicador asíncrono con retries + DLQ
- Eventos versionados (schema evolution)

### 9.2 Event catalog (obligatorio)
- `EventName` + versión
- producer + consumer(s)
- payload schema + PII flags
- retry policy + idempotency rules

**Gate EDA (bloquea):**
- Publicar evento fuera de transacción sin outbox en flows críticos.
- Consumidores sin idempotencia.

---

## 10) Test Strategy (mínimos exigibles)
- **Unit:** dominio (VOs, invariantes) + reglas core
- **Integration:** casos de uso con DB (o repos) + idempotencia
- **Contract:** integraciones externas (mocks verificados)
- **Security:** tests básicos de authz (role/tenant)
- **Migrations:** smoke test + rollback plan

---

## 11) Formato obligatorio de salida (cuando se active este skill)
Windsurf debe responder con:

1) **Backend Profile**  
2) **API Contract** (endpoints/queries, payloads, errores, paginación, versionado)  
3) **Architecture Plan** (bounded contexts, capas hexagonales, CQRS/EDA si aplica)  
4) **AuthN/AuthZ Plan** (OIDC, roles, policies, MFA opcional)  
5) **Multi-tenant Plan** (aislamiento, tenant scoping, límites, planes)  
6) **Data Model Plan** (DB schema + índices + migraciones)  
7) **Ledger Plan** (si aplica) + invariantes + reconciliación  
8) **Jobs/Queue Plan** (tipos, idempotencia, retries, DLQ, métricas)  
9) **Observability Plan** (OTel, logs, métricas, health checks)  
10) **Next Steps** (accionables)

---

## 12) Señales de deuda backend (Windsurf debe advertir)
- APIs sin versión, sin paginación o sin errores estándar.
- Writes sin idempotencia en flujos críticos.
- Multi-tenant “a medias” (queries sin scope).
- AuthZ débil (solo RBAC superficial, sin policies).
- Ledger “anémico” sin balanceo ni reversals.
- Migraciones peligrosas (sin expand/contract).
- Jobs sin DLQ ni idempotencia.
- Sin observabilidad (tracing/metrics/logs estructurados).

---
**End of skill.**
