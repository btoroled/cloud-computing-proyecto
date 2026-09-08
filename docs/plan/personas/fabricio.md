# Fabricio — MS4 + MS5 + Data Science ⚠️

**Repos:** `ms4-manifiesto-api`, `ms5-analitica-api`, `aeropuerto-data-science`
**Stack:** Python + FastAPI (MS4 sin BD; MS5 + boto3 → Athena) + Python para `seeds/` e ingesta.
**Carga estimada:** ~14 d — la más alta del equipo. Ver [rebalanceo §5](../distribucion-trabajo.md#5-cuellos-de-botella-y-rebalanceo).

Fuentes: [backend.md](../backend.md) §7, §8, §3 · [data-science.md](../data-science.md) ·
[exposiciones-e-informe.md](../exposiciones-e-informe.md).

> **Estás en el camino crítico del proyecto:** `seeds/` → carga 20k → ingesta → S3 → Glue → Athena →
> MS5 → Dashboard. Cualquier atraso tuyo empuja a Alexander.

---

## F0 — Setup y contratos (Mié 2 – Vie 4)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-TX-03 | `openapi.yaml` borrador de MS4 y MS5 (contract-first) | 0.5d | BE-TX-01 | OpenAPI válido, revisado en standup Día 2 |
| ☐ | DS-01 | Estructura del repo `aeropuerto-data-science` (`seeds/`, `ingesta/`, `glue/`, `athena/`) | 0.25d | — | Repo creado desde plantilla |
| ☐ | DS-02 | `seeds/` v1: generador con `SEED` fijo, orden **MS2→MS1→MS3**, rangos de ID del [plan §3](../plan-de-trabajo.md#3-datos-de-prueba-integridad-entre-servicios) | 1.5d | enums | Genera CSV coherentes entre los 3 dominios |
| ☐ | DS-03 | Escribir las 5 consultas (Q1–Q5) en **SQL sobre Postgres local** para validar la lógica | 1d | modelo | Las 5 devuelven resultados razonables en local |
| ☐ | MS4-01 | Scaffold FastAPI + `Dockerfile` (sin BD) + config de URLs de MS1/MS2/MS3 por entorno | 0.5d | BE-TX-04 | `GET /health/dependencias` reporta estado de los 3 |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-TX-09 | Cliente HTTP compartido para llamadas entre servicios (timeout, reintento, propagación de error) | 0.5d | BE-TX-08 | Usado por MS1, MS3, MS4 |
| ☐ | MS4-02 | `GET /manifiesto/{vuelo_id}` — agrega vuelo+aeronave+aerolínea (MS2), pasajeros+ticket+checkin+equipaje (MS1), tripulación (MS2), recursos+incidencias abiertas (MS3) | 1.5d | MS1-06, MS2-05/08, MS3-05/07, BE-TX-09 | Respuesta consolidada; vuelo inexistente → 404 |
| ☐ | MS5-01 | Scaffold FastAPI + `Dockerfile` + config Athena (workgroup, output S3, región) con `LabRole` | 0.5d | BE-TX-04 | `GET /health` OK; credenciales del lab cargadas |
| ☐ | DS-06 (apoyo) | Apoyar a Guillermo en `ingesta-ms1` | — | DS-04 | `ingesta-ms1` cargando a S3 |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

### MS4
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS4-03 | `GET /manifiesto/{vuelo_id}/pasajeros` y `/resumen` (conteos: pax, kg equipaje, incidencias abiertas) | 0.5d | MS4-02 | Subrecursos coherentes |
| ☐ | MS4-04 | Manejo de dependencia caída (timeout / 502) → respuesta parcial con `warnings[]` | 0.5d | MS4-02 | Si MS3 cae, devuelve resto + warning |
| ☐ | MS4-05 | Swagger + ejemplos + pruebas con mocks de MS1/MS2/MS3 | 0.5d | MS4-02 | `/docs` completo; pruebas verdes |
| ☐ | MS4-06 | README + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | Imagen publicada |

### Data Science
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | DS-09 | Glue: database `aeropuerto_lake` + 1 crawler por prefijo `raw/msX/` | 0.75d | DS-06..08 | Tablas visibles en el catálogo |
| ☐ | DS-10 | Revisión/ajuste de esquemas inferidos por el crawler (tipos, `timestamp`, columnas) | 0.75d | DS-09 | `SELECT * LIMIT 10` correcto por tabla (R9) |
| ☐ | DS-11 | Implementar Q1–Q5 en **Athena** (workgroup + output S3) | 1.5d | DS-10 | Las 5 corren y devuelven filas |
| ☐ | DS-12 | Crear las 2 **vistas** `vw_recaudacion_tuua` y `vw_retrasos_hora_punta` | 0.5d | DS-11 | `SHOW VIEWS` las lista; DDL en `athena/` |
| ☐ | DS-13 | **E/R del catálogo** (`diagramas/er-catalogo-datalake.drawio`) con todas las tablas + claves de join (con Alexander) | 1d | DS-09 | Diagrama entregado |
| ☐ | DS-14 | Evidencias en `docs/evidencias/athena/` (capturas de las 5 queries + 2 vistas + `aws s3 ls` + Glue console) | 0.5d | DS-11..13 | Carpeta de evidencias completa |

### MS5
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS5-02 | Capa de ejecución Athena (lanzar query, *poll*, leer resultados, cachear por TTL) | 0.75d | DS catálogo | Ejecuta una query de prueba y devuelve filas |
| ☐ | MS5-03 | `GET /analitica/recursos-mas-fallas?dias=` | 0.5d | MS5-02, DS Q1 | Join recurso×incidencia |
| ☐ | MS5-04 | `GET /analitica/retraso-promedio?tipo=` | 0.5d | MS5-02, DS Q2 | Promedio en minutos por tipo |
| ☐ | MS5-05 | `GET /analitica/incidencias-combustible-por-aerolinea` | 0.5d | MS5-02, DS Q3 | Ranking por aerolínea |
| ☐ | MS5-06 | `GET /analitica/recaudacion-tuua-por-categoria` (lee `vw_recaudacion_tuua`) | 0.5d | MS5-02, DS vista 1 | Monto por categoría |
| ☐ | MS5-07 | `GET /analitica/vuelos-hora-punta-retrasados` (lee `vw_retrasos_hora_punta`) | 0.5d | MS5-02, DS vista 2 | % de vuelos hora punta retrasados |
| ☐ | MS5-08 | Swagger + pruebas (mock del cliente Athena) | 0.5d | MS5-03..07 | `/docs` completo; pruebas verdes |
| ☐ | MS5-09 | README + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | Imagen publicada |

### Frontend (apoyo)
| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | FE-10 | **Dashboard de crisis**: aportar consultas/formato de datos de MS5; Alexander maqueta | 2d (compartido) | MS5-03..07 | 5 tarjetas/gráficos con datos reales de Athena |

---

## Entregas por hito

- **Hito 1:** MS4 devolviendo manifiesto simple; `seeds/` operativo; 5 queries validadas en local.
- **Hito 2:** MS5 con 5 endpoints Athena; Glue + 2 vistas; E/R del catálogo; sección de transformaciones.

## Informe

- **Sección 3 (Backend)** — MS4 y MS5.
- **Sección 4 — Transformaciones del modelo monolítico** (jerarquía `Persona` partida, FK suaves).
- **Sección 6 — Data Science** (`aws s3 ls` · Glue console · E/R del catálogo · 4 consultas Athena · 2 vistas).

## Rebalanceo disponible si te saturas (aplicar en el standup)

1. **`seeds/`**: que cada dev genere el seed de su BD; tú solo defines el contrato de IDs y orquestas (−1d).
2. **E/R del catálogo (DS-13)**: pasa a Alexander (es diagramación) (−1d).
3. **MS5**: si Guillermo o Edinson terminan antes de F2, uno toma MS5 (Python + boto3) (−3d).
4. **Dashboard (FE-10)**: ya es compartido con Alexander.

## Riesgos a tu cargo

- **R8** `persona` duplicada (IsA partida) → documentar la transformación en el informe con E/R por servicio.
- **R9** Crawlers de Glue infieren tipos mal → `CREATE TABLE` explícito o Parquet; validar con `SELECT ... LIMIT 10`.
- **R5** (con Guillermo/Mariano/Edinson) IDs que no cruzan → `seeds/` orden MS2→MS1→MS3 con `SEED` fijo; DS-03 valida joins en local.

## Nota de estado

- `aeropuerto-data-science`: ✅ `seeds/` con generadores MS1/MS2/MS3 (DS-02 avanzado). Faltan `ingesta/`, `glue/`, `athena/`. El `README.md` del repo aún usa "Dev A/B/C/D" — actualizar a nombres.
- `ms4-manifiesto-api` y `ms5-analitica-api`: ⚠️ solo `initial commit` (LICENSE + `.gitignore` + README de una línea). Sin scaffold aún.

## Con todo el equipo

- **BE-TX-01** — acordar `contratos/enums.md` y rangos de ID (F0).
- **EX-04** — exposición virtual con ACL (F1).
- **EX-11** — ensayo general de la demo el Vie 19 (F2).
- **EX-12** — exposición presencial + demo en vivo (F3, **obligatoria**).
