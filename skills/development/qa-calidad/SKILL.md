---
name: qa-calidad
description: Skill de QA y Calidad para prevenir regresiones y errores de datos. Define estrategias de pruebas (unit/integration/e2e/contract), automatización web y mobile, validación de consistencia numérica, y gestión de releases con staging y rollout gradual.
---

# ✅ Windsurf Skill — QA & Calidad (No romper finanzas)
**Skill ID:** SK-QA-001  
**Aplica a:** Todos los verticals; **crítico** en fintech/PFM, retail (órdenes/pagos), health/med (registros), legal/reg (evidencia)  
**Objetivo:** prevenir regresiones y errores de datos mediante una estrategia de pruebas completa (unit/integration/e2e/contract), automatización (web + mobile), validación de consistencia numérica, y release management con staging + rollout gradual.

---

## 0) QA Profile (output obligatorio)
Antes de diseñar/codificar, Windsurf debe fijar:

- **Criticality:** C0 (baja) | C1 (media) | C2 (alta) | C3 (crítica)
- **Platforms:** API | Web | Mobile
- **Test levels:** unit | integration | e2e | contract (seleccionados)
- **Tooling:** Jest/Vitest | Playwright/Cypress | Detox/Appium | Pact | otro
- **Environments:** dev/staging/prod + datos de prueba
- **Release strategy:** canary/blue-green/phased rollout + feature flags
- **Data correctness scope:** balances, budgets, aggregates, ledger (sí/no)
- **Risk Tier:** R0–R3

> Gate: sin QA Profile explícito, Windsurf **no avanza**; declara supuestos “Hard” + impacto.

---

## 1) Principios (no negociables)
1. **Calidad de datos > UI bonita:** un número incorrecto es un bug crítico.
2. **Shift-left:** tests se escriben junto al código, no al final.
3. **Piramide de pruebas:** unit amplia; integration suficiente; e2e mínima y estratégica.
4. **Determinismo:** tests repetibles (control de tiempo, seeds, fixtures).
5. **Contratos estables:** APIs y eventos deben tener pruebas de contrato.
6. **Release seguro:** staging + gates + rollout gradual para cambios críticos.

---

## 2) Estrategia de pruebas (unit / integration / e2e / contract)
### 2.1 Unit tests (obligatorios)
- Dominio: value objects, invariantes, reglas de negocio
- Cálculos: sumas, saldos, presupuestos, redondeos, periodos
- Parsers/mappers críticos

**Gate unit (bloquea):**
- Feature core sin unit tests para invariantes/cálculos.

### 2.2 Integration tests (obligatorios en flows críticos)
- Casos de uso completos con DB (y cola si aplica)
- Idempotencia (reintentos con misma key)
- Multi-tenant scoping (tenant isolation)
- Migraciones (smoke upgrade)

**Gate integration (bloquea):**
- Writes críticos sin integración con persistencia real o simulación robusta.

### 2.3 E2E tests (mínimo estratégico)
- “Happy path” principal por plataforma:
  - onboarding/login
  - acción core (ej. crear transacción, ver reporte)
  - export (si aplica)
- Smoke suite (rápida) para cada release
- Regression suite (más amplia) por nightly/semana

**Gate e2e (bloquea para C2/C3):**
- No hay smoke tests automatizados.
- No hay suite mínima por plataforma afectada.

### 2.4 Contract testing (obligatorio con integraciones)
- API provider/consumer (REST/GraphQL)
- Webhooks/eventos (schema versioned)
- Integraciones bancarias/pagos (si aplica)

**Gate contract (bloquea):**
- Cambios de API/eventos sin pruebas de contrato.

---

## 3) QA Automation (web + mobile) y suites
### 3.1 Web automation
- Playwright (recomendado) o Cypress
- Tests orientados a flujos, no a detalles frágiles
- Selectores estables (`data-testid`)

### 3.2 Mobile automation
- Detox (Flutter: integración vía adaptaciones) o Appium
- Pruebas en dispositivos emulados + al menos 1 real en CI (si posible)
- Permisos/notifications mockeadas

### 3.3 Suites recomendadas
- **Smoke (PR/merge):** < 10 min, flujos críticos
- **Regression (nightly):** cobertura amplia
- **Performance smoke (semanal):** tiempo de carga, listas grandes
- **Data correctness suite (siempre):** invariantes numéricas

**Gate automation (bloquea en C2+):**
- No existe smoke suite automatizada.
- Regression inexistente o solo manual.

---

## 4) Data Correctness Testing (finanzas/ledger/reporting)
> Obligatorio cuando el producto calcula saldos, presupuestos, reportes o ledger.

### 4.1 Invariantes numéricas (obligatorio)
- Sumatorias por periodo = total mostrado
- Saldos coherentes con transacciones (y con ajustes/reversals si aplica)
- Presupuesto: consumido + restante = asignado
- Split transactions: suma de splits = monto total
- Multi-currency: reglas explícitas (conversión, tasa, redondeo)

### 4.2 Golden datasets (recomendado)
- Fixtures con escenarios reales:
  - devoluciones/reembolsos
  - transacciones duplicadas (dedupe)
  - importaciones parciales
  - cambios de categoría retroactivos
  - periodos cruzados (fin de mes, zona horaria)

### 4.3 Ledger (si aplica)
- Doble partida balanceada (debits = credits)
- No mutar entries posteadas; usar reversals
- Reconciliación reproducible

**Gate data correctness (bloquea):**
- Cambios en cálculos/reporting sin tests sobre datasets “golden”.
- Falta de estrategia de rounding/periodos/timezone.

---

## 5) Gestión de releases (versionado, staging, rollout)
### 5.1 Reglas duras
- **Staging obligatorio** para C1+
- **Checklist de release**:
  - smoke tests OK
  - migraciones safe (expand/contract)
  - observabilidad activa (crash/error/latency)
  - feature flags para cambios riesgosos
- **Rollout gradual** (canary/phased):
  - 5% → 25% → 50% → 100% (ejemplo)
- **Rollback plan**:
  - app rollback + DB forward-safe (no “down migrations” en caliente)

### 5.2 Gates de release
- Para C2/C3:
  - regression mínima (nightly latest) + smoke green
  - pruebas de contrato green
  - monitoreo post-release (1–2h) con alertas

**Gate release (bloquea):**
- Deploy directo a prod sin staging.
- Cambios críticos sin feature flag.
- Sin plan de rollback y monitoreo.

---

## 6) Defect management y calidad operacional
- Clasificación de severidad:
  - **S0:** seguridad/privacidad, pérdida de datos
  - **S1:** números incorrectos, duplicación, integridad financiera
  - **S2:** flujo roto, crashes
  - **S3:** UI/UX menor
- SLA de corrección (según equipo/riesgo)
- Postmortems para S0/S1 en C2/C3

---

## 7) Outputs obligatorios (por fase BMAD)
### BRIEF
- QA Profile + criticality + riesgos + supuestos

### MODEL
- Test pyramid por módulo
- Contract boundaries (providers/consumers)
- Data correctness invariants + datasets

### ACTION
- Lista de tests a crear (unit/integration/e2e/contract)
- Suites (smoke/regression) + tiempos objetivo
- Datos de prueba/fixtures + seeds
- Estrategia de mocking (tiempo, terceros, notificaciones)

### DEPLOY
- Release checklist + gates + rollout plan
- Monitoreo post-release + criterios de rollback

---

## 8) Formato obligatorio de salida (cuando se active este skill)
Windsurf debe responder con:

1) **QA Profile**  
2) **Test Strategy** (unit/integration/e2e/contract) + pirámide  
3) **Automation Plan** (web/mobile) + suites (smoke/regression)  
4) **Data Correctness Plan** (invariantes + golden datasets)  
5) **Release Plan** (staging, rollout, rollback, post-monitoring)  
6) **Next Steps** (accionables)

---

## 9) Señales de deuda de QA (Windsurf debe advertir)
- Tests solo e2e (frágiles) sin unit/integration.
- Cambios de cálculos sin golden datasets.
- Sin contract tests en integraciones.
- Staging inexistente o smoke manual.
- Rollout “big bang” sin flags.
- Falta de determinismo (tests flakey por tiempo/datos).

---
**End of skill.**
