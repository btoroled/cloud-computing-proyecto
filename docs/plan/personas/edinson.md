# Edinson — MS3 Infraestructura / Incidencias

**Repo:** `ms3-infraestructura-api` · **Stack:** Node.js + Express + MongoDB 7 · **Ingesta:** `ingesta-ms3`
**Carga estimada:** ~10.5 d

Fuentes: [backend.md](../backend.md) §6 · [data-science.md](../data-science.md) §5 ·
[exposiciones-e-informe.md](../exposiciones-e-informe.md) · [enums](../../contratos/enums.md).

---

## F0 — Setup y contratos (Mié 2 – Vie 4)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-TX-03 | `openapi.yaml` borrador de MS3 (contract-first) | 0.5d | BE-TX-01 | OpenAPI válido, revisado en standup Día 2 |
| ☐ | MS3-01 | Scaffold Express + `Dockerfile` + `compose` (app + mongo) + healthcheck | 0.5d | BE-TX-04 | `GET /health` OK |
| ☐ | MS3-02 | JSON Schema de `recursos` (manga/radar), `incidencias`, `asignaciones` + validación con `ajv` | 1d | BE-TX-01 | Schemas en `docs/er/ms3-mongo-schemas.md` |
| ☐ | — | `docker compose up` local del servicio + su BD, con validación y 1 endpoint real | — | MS3-01/02 | Servicio arranca con datos |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS3-03 | CRUD `recursos` (manga\|radar) + `GET /recursos?tipo=&estado=` | 0.5d | MS3-02 | Filtro por estado/tipo |
| ☐ | MS3-04 | `PATCH /recursos/{id}/estado` (Libre/Ocupado/Mantenimiento) | 0.25d | MS3-03 | Estado inválido → 422 |
| ☐ | MS3-05 | `POST /incidencias` con **validación de vuelos contra MS2** + `GET /incidencias` + filtros `?tipo=&desde=&hasta=` | 1d | MS2-07, BE-TX-09 | `vuelo_id` inexistente → 422; log de llamada |
| ☐ | MS3-06 | `GET /incidencias/{id}`, `PATCH /incidencias/{id}/cierre` (setea `fecha_cierre`) | 0.5d | MS3-05 | Incidencia cerrada no se re-cierra |
| ☐ | MS3-07 | `POST /asignaciones` (vuelo↔recurso) + `GET /asignaciones?vuelo_id=` / `?recurso_id=` + regla clase manga ≥ clase aeronave | 0.75d | MS3-03, MS2-07 | Acople de clase mayor a manga menor → 422 |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS3-08 | Generador + **carga masiva ≥20 000** en `incidencias` (una vez) | 1d | seeds compartido | `db.incidencias.countDocuments()` ≥ 20000 |
| ☐ | MS3-09 | Swagger (`swagger-ui-express`) + pruebas (`jest` + `supertest`) | 0.5d | MS3-03..07 | `/docs` completo; pruebas verdes |
| ☐ | MS3-10 | README + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | Imagen publicada |
| ☐ | DS-08 | **`ingesta-ms3`**: leer colecciones + **aplanar `incidencias`** en `incidencia` / `incidencia_recurso` / `incidencia_vuelo` → `raw/ms3/` | 1.5d | DS-04, MS3-02 | 5 archivos en S3; conteos cuadran |
| ☐ | DS-15 | Carga masiva real: ejecutar `ingesta-ms3` **una vez** tras los 20k (con Guillermo y Mariano) | 0.5d (repartido) | Backend F2 (20k) | S3 con el 100% de MS3 |
| ☐ | — | JSON Schema de Mongo consolidado en `docs/er/ms3-mongo-schemas.md` | — | MS3-02 | Documento entregado |
| ☐ | EX-05/06 | Poblar `docs/evidencias/backend/` + redactar tu sección del informe | 0.5d | verificación | Sección sin "TODO" |

---

## Entregas por hito

- **Hito 1:** MS3 con CRUD mínimo + MS3→MS2 + BD MongoDB conectada.
- **Hito 2:** MS3 completo + 20k en `incidencias` + Swagger + JSON Schema + `ingesta-ms3`.

## Informe

- **Sección 3 (Backend)** — tu microservicio: Swagger, `countDocuments()` ≥ 20 000, log de consumo MS3→MS2.
- **Sección 3b** — JSON Schema de MongoDB.

## Dependencias clave

- **Necesitas de Mariano:** `MS2-07 /vuelos/{id}/exists` para MS3-05 y MS3-07.
- **Necesitas de Benja:** `BE-TX-09` cliente HTTP; `DS-04` bucket S3.
- **Necesitas de Fabricio:** `seeds/` con contrato de IDs.
- **Te esperan:** Fabricio (MS4 `/manifiesto` consume MS3-05/07).

## Apoyo que puedes dar

- Si MS3 va adelantado, **ayudar a Mariano** con el generador de `asiento` / `opera_tripulacion` (tablas de volumen de MS2).

## Nota de estado

✅ El repo `ms3-infraestructura-api` ya existe con scaffold Express + Mongo + `Dockerfile` + `compose` +
`GET /health`. Falta: JSON schemas, CRUD, `openapi.yaml`, CI a GHCR, README real, tests. `package.json`
declara `license: ISC` — cambiar a **MIT** para alinear con la convención del proyecto.

## Con todo el equipo

- **BE-TX-01** — acordar `contratos/enums.md` y rangos de ID (F0).
- **EX-04** — exposición virtual con ACL (F1).
- **EX-11** — ensayo general de la demo el Vie 19 (F2).
- **EX-12** — exposición presencial + demo en vivo (F3, **obligatoria**).
