# Fabricio — MS4 + MS5 + Data Science ⚠️

**Repos:** `ms4-manifiesto-api`, `ms5-analitica-api`, `aeropuerto-data-science`
**Stack:** Python + FastAPI (MS4 sin BD; MS5 + boto3 → Athena) + Python para `seeds/` e ingesta.
**Carga estimada:** ~14 d — la más alta del equipo. Ver [rebalanceo §5](../distribucion-trabajo.md#5-cuellos-de-botella-y-rebalanceo).

Fuentes: [backend.md](../backend.md) §7, §8, §3 · [data-science.md](../data-science.md) ·
[exposiciones-e-informe.md](../exposiciones-e-informe.md).

> **Estás en el camino crítico del proyecto:** `seeds/` → carga 20k → ingesta → S3 → Glue → Athena →
> MS5 → Dashboard. Cualquier atraso tuyo empuja a Alexander.

---

## F0 — Setup y contratos (Mié 2 – Vie 4) — ✅ CERRADA

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☑ | BE-TX-03 | `openapi.yaml` borrador de MS4 y MS5 (contract-first) | 0.5d | BE-TX-01 | ✅ OpenAPI válido en `/openapi.json` de ambos + ejemplos en `/docs` |
| ☑ | DS-01 | Estructura del repo `aeropuerto-data-science` (`seeds/`, `ingesta/`, `glue/`, `athena/`) | 0.25d | — | ✅ Repo creado, README con estructura |
| ☑ | DS-02 | `seeds/` v1: generador con `SEED` fijo, orden **MS2→MS1→MS3**, rangos de ID del [plan §3](../plan-de-trabajo.md#3-datos-de-prueba-integridad-entre-servicios) | 1.5d | enums | ✅ CSV/JSONL coherentes, reproducible byte-a-byte (SEED=20260905, verificado con MD5) |
| ☑ | DS-03 | Escribir las 5 consultas (Q1–Q5) en **SQL sobre Postgres local** para validar la lógica | 1d | modelo | ✅ Las 5 devuelven resultados; outputs en `athena/expected/` |
| ☑ | MS4-01 | Scaffold FastAPI + `Dockerfile` (sin BD) + config de URLs de MS1/MS2/MS3 por entorno | 0.5d | BE-TX-04 | ✅ `GET /health/dependencias` chequea MS1/2/3 en paralelo con `asyncio.gather` |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1) — ✅ CERRADA

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☑ | BE-TX-09 | Cliente HTTP compartido para llamadas entre servicios (timeout, reintento, propagación de error) | 0.5d | BE-TX-08 | ✅ `app/clients/base.py` en MS4 con retries exponenciales |
| ☑ | MS4-02 | `GET /manifiesto/{vuelo_id}` — agrega vuelo+aeronave+aerolínea (MS2), pasajeros+ticket+checkin+equipaje (MS1), tripulación (MS2), recursos+incidencias abiertas (MS3) | 1.5d | MS1-06, MS2-05/08, MS3-05/07, BE-TX-09 | ✅ Respuesta consolidada; vuelo inexistente → 404; probado end-to-end en AWS con MS1/MS2/MS3 reales (data del seed: LA0032 TRU→LIM, Copa Airlines, tripulación real) |
| ☑ | MS5-01 | Scaffold FastAPI + `Dockerfile` + config Athena (workgroup, output S3, región) con `LabRole` | 0.5d | BE-TX-04 | ✅ `/health` OK; env vars ATHENA_* documentadas |
| ☑ | DS-06 (apoyo) | Apoyar a Guillermo en `ingesta-ms1` | — | DS-04 | ✅ Guillermo lo cerró; PR mergeado en `aeropuerto-data-science/ingesta/ingesta-ms1/` |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

### MS4 — ✅ CÓDIGO CERRADO (tag `v1.0` pendiente tras merge)
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☑ | MS4-03 | `GET /manifiesto/{vuelo_id}/pasajeros` y `/resumen` (conteos: pax, kg equipaje, incidencias abiertas) | 0.5d | MS4-02 | ✅ Subrecursos coherentes, tests verdes |
| ☑ | MS4-04 | Manejo de dependencia caída (timeout / 502) → respuesta parcial con `warnings[]` | 0.5d | MS4-02 | ✅ Si MS3 cae, devuelve resto + warning; verificado en AWS (probado con `curl` — MS4 devolvió `warnings: ["MS1/tickets: 404..."]` al bajar MS1) |
| ☑ | MS4-05 | Swagger + ejemplos + pruebas con mocks de MS1/MS2/MS3 | 0.5d | MS4-02 | ✅ `/docs` con ejemplos completo/degraded/404; 7 tests con `httpx.MockTransport` |
| ⏳ | MS4-06 | README + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | 🟡 README v1.0 pusheado (PR `chore/v1-readme-ejemplos`). **Falta:** tras merge, `git tag v1.0 && git push origin v1.0` dispara GHCR |

### Data Science
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | DS-09 | Glue: database `aeropuerto_lake` + 1 crawler por prefijo `raw/msX/` | 0.75d | DS-06..08 | 🔴 Bloqueado por credenciales AWS |
| ☐ | DS-10 | Revisión/ajuste de esquemas inferidos por el crawler (tipos, `timestamp`, columnas) | 0.75d | DS-09 | 🔴 Bloqueado por DS-09 |
| ⏳ | DS-11 | Implementar Q1–Q5 en **Athena** (workgroup + output S3) | 1.5d | DS-10 | 🟡 Los 5 `.sql` en sintaxis Athena/Trino listos en `athena/queries/athena/` (PR `feature/ds-11-athena-sql`). **Falta ejecutar en AWS** — bloqueado por DS-10 |
| ⏳ | DS-12 | Crear las 2 **vistas** `vw_recaudacion_tuua` y `vw_retrasos_hora_punta` | 0.5d | DS-11 | 🟡 DDL listo en `athena/queries/views/` (mismo PR de DS-11). **Falta ejecutar `CREATE VIEW`** en Athena — bloqueado por DS-11 |
| ☐ | DS-13 | **E/R del catálogo** (`diagramas/er-catalogo-datalake.drawio`) con todas las tablas + claves de join (con Alexander) | 1d | DS-09 | ⏳ Borrador Mermaid en preparación para Alexander |
| ☐ | DS-14 | Evidencias en `docs/evidencias/athena/` (capturas de las 5 queries + 2 vistas + `aws s3 ls` + Glue console) | 0.5d | DS-11..13 | 🔴 Bloqueado por DS-11/12 |

### MS5 — ✅ CÓDIGO CERRADO (tag `v1.0` pendiente tras merge)
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☑ | MS5-02 | Capa de ejecución Athena (lanzar query, *poll*, leer resultados, cachear por TTL) | 0.75d | DS catálogo | ✅ `AthenaClient` boto3 con start → poll (500ms, timeout 60s) → get_results paginado + cache SHA-256 con TTL. Contra AWS real funciona sin cambios (probado con mocks) |
| ☑ | MS5-03 | `GET /analitica/recursos-mas-fallas?dias=` | 0.5d | MS5-02, DS Q1 | ✅ Query Q1 portada a Athena; validación `dias 1-365` |
| ☑ | MS5-04 | `GET /analitica/retraso-promedio?tipo=` | 0.5d | MS5-02, DS Q2 | ✅ Query Q2; validación `tipo ∈ {Nacional, Internacional}` con Pydantic `Literal` |
| ☑ | MS5-05 | `GET /analitica/incidencias-combustible-por-aerolinea` | 0.5d | MS5-02, DS Q3 | ✅ Query Q3 con tasa por 1000 vuelos |
| ☑ | MS5-06 | `GET /analitica/recaudacion-tuua-por-categoria` (lee `vw_recaudacion_tuua`) | 0.5d | MS5-02, DS vista 1 | ✅ Endpoint funcional; consumirá la vista tras DS-12 |
| ☑ | MS5-07 | `GET /analitica/vuelos-hora-punta-retrasados` (lee `vw_retrasos_hora_punta`) | 0.5d | MS5-02, DS vista 2 | ✅ Endpoint funcional; consumirá la vista tras DS-12 |
| ☑ | MS5-08 | Swagger + pruebas (mock del cliente Athena) | 0.5d | MS5-03..07 | ✅ `/docs` con ejemplos; 16 tests con `MagicMock` boto3 |
| ⏳ | MS5-09 | README + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | 🟡 README v1.0 pusheado (PR `feature/ms5-athena-real`). **Falta:** tras merge, `git tag v1.0 && git push origin v1.0` |

### Frontend (apoyo)
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | FE-10 | **Dashboard de crisis**: aportar consultas/formato de datos de MS5; Alexander maqueta | 2d (compartido) | MS5-03..07 | ⏳ Endpoints MS5 listos con ejemplos OpenAPI para Alexander. Bloqueado por MS5 funcional en AWS |

---

## Entregas por hito

- **Hito 1:** ✅ MS4 devolviendo manifiesto simple; `seeds/` operativo; 5 queries validadas en local.
- **Hito 2:** MS5 con 5 endpoints Athena ✅ código listo; Glue + 2 vistas 🟡 SQL listo; E/R del catálogo ⏳; sección de transformaciones ⏳ (informe).

## Informe

- **Sección 3 (Backend)** — MS4 y MS5.
- **Sección 4 — Transformaciones del modelo monolítico** (jerarquía `Persona` partida, FK suaves).
- **Sección 6 — Data Science** (`aws s3 ls` · Glue console · E/R del catálogo · 4 consultas Athena · 2 vistas).

## Rebalanceo disponible si te saturas (aplicar en el standup)

1. **`seeds/`**: que cada dev genere el seed de su BD; tú solo defines el contrato de IDs y orquestas (−1d). ✅ Aplicado — cada dev clona el repo y corre `python seeds/generar.py`.
2. **E/R del catálogo (DS-13)**: pasa a Alexander (es diagramación) (−1d). ⏳ En curso: borrador Mermaid a compartir.
3. **MS5**: si Guillermo o Edinson terminan antes de F2, uno toma MS5 (Python + boto3) (−3d). ❌ No aplicado — MS5 lo hice yo.
4. **Dashboard (FE-10)**: ya es compartido con Alexander.

## Riesgos a tu cargo

- **R8** `persona` duplicada (IsA partida) → documentar la transformación en el informe con E/R por servicio.
- **R9** Crawlers de Glue infieren tipos mal → `CREATE TABLE` explícito o Parquet; validar con `SELECT ... LIMIT 10`.
- **R5** (con Guillermo/Mariano/Edinson) IDs que no cruzan → ✅ mitigado con `seeds/` orden MS2→MS1→MS3 con `SEED` fijo; DS-03 valida joins en local.

## Nota de estado (2026-09-19)

**Código listo y pusheado como PR (esperando merge):**
- `ms4-manifiesto-api` — `chore/v1-readme-ejemplos` (README v1.0 + ejemplos OpenAPI)
- `ms5-analitica-api` — `feature/ms5-athena-real` (AthenaClient real + 5 endpoints Q1-Q5 + README v1.0 + 16 tests)
- `aeropuerto-data-science` — `feature/ds-11-athena-sql` (5 SQL Athena + 2 vistas + `SETUP_ATHENA.md`)

**Post-merge (tú, mismo día):**
```bash
cd ms4-manifiesto-api && git tag v1.0 && git push origin v1.0
cd ../ms5-analitica-api && git tag v1.0 && git push origin v1.0
```
→ dispara `build-push-ghcr.yml` → publica `ghcr.io/cloud-mla/{ms4-manifiesto-api,ms5-analitica-api}:v1.0` y `:latest`.

**Bloqueado por AWS (Benja):**
- DS-09 (Glue crawlers) y DS-10 (ajuste de schemas) — necesitan `Start Lab` + acceso al S3 con datos de ingesta ya cargados.
- DS-14 (evidencias Athena) — depende de DS-11/12 ejecutados.
- FE-10 (Dashboard) — depende de MS5 funcionando en AWS.

**En curso:**
- DS-13 (E/R del catálogo, `.drawio`) — borrador Mermaid en preparación para compartir con Alexander.

## Con todo el equipo

- **BE-TX-01** — acordar `contratos/enums.md` y rangos de ID (F0). ✅ Cerrado.
- **EX-04** — exposición virtual con ACL (F1). ✅ Cerrado (Hito 1 aprobado; ACL dio plazo extra hasta 8PM del mismo día para arreglar swagger — el ajuste queda pendiente de deploy en AWS).
- **EX-11** — ensayo general de la demo el Vie 19 (F2).
- **EX-12** — exposición presencial + demo en vivo (F3, **obligatoria**).
