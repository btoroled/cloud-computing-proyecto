# Guillermo — MS1 Pasajeros / Tickets

**Repo:** `ms1-pasajeros-api` · **Stack:** Python + FastAPI + MySQL 8 · **Ingesta:** `ingesta-ms1`
**Carga estimada:** ~10 d

Fuentes: [backend.md](../backend.md) §4 · [data-science.md](../data-science.md) §5 ·
[exposiciones-e-informe.md](../exposiciones-e-informe.md) · [enums](../../contratos/enums.md).

---

## F0 — Setup y contratos (Mié 2 – Vie 4)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-TX-03 | `openapi.yaml` borrador de MS1 (contract-first) | 0.5d | BE-TX-01 | OpenAPI válido, revisado en standup Día 2 |
| ☐ | MS1-01 | Scaffold FastAPI + `Dockerfile` + `compose` local (app + mysql) | 0.5d | BE-TX-04 | `GET /health` responde en local |
| ☐ | MS1-02 | Modelo y migraciones: `persona`, `pasajero`, `categoria_migratoria`, `ticket`, `checkin`, `equipaje` | 1d | BE-TX-01 | Migración aplica; E/R exportado a `docs/er/ms1-mysql-er` |
| ☐ | — | `docker compose up` local del servicio + su BD, con migraciones y 1 endpoint real | — | MS1-01/02 | Servicio arranca con datos |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS1-03 | CRUD `pasajeros` + búsqueda por clave alterna (`tipo_documento`+`numero_documento`) | 0.5d | MS1-02 | Endpoints §4.1 OK |
| ☐ | MS1-04 | `GET /categorias-migratorias` + carga de las 3 categorías con tarifa TUUA | 0.25d | MS1-02 | Devuelve Nacional/Internacional/Transito |
| ☐ | MS1-05 | `POST /tickets` con **validación de vuelo contra MS2** (`GET /vuelos/{id}/exists`) | 1d | MS2-07, BE-TX-09 | Vuelo inexistente/cancelado → 422; log de llamada saliente |
| ☐ | MS1-06 | `GET /tickets/{id}`, `GET /tickets?vuelo_id=`, `GET /pasajeros/{id}/tickets` | 0.5d | MS1-05 | Filtros funcionan |
| ☐ | MS1-07 | `POST /tickets/{id}/checkin` (1:1 con ticket) | 0.5d | MS1-05 | Segundo check-in del mismo ticket → 409 |
| ☐ | MS1-08 | `POST /equipajes` + `GET /equipajes?pasajero_id=` / `?vuelo_id=` | 0.5d | MS1-02 | Tag ID único; peso válido |
| ☐ | DS-06 | **`ingesta-ms1`**: conectar MySQL, `SELECT *` 100% por tabla, escribir CSV, `put_object` a `raw/ms1/` | 1d | DS-04, MS1-02 | Archivos de las 6 tablas en S3 |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS1-09 | Generador de datos + **carga masiva ≥20 000** en `ticket` y `equipaje` (una vez) | 1d | seeds compartido | `SELECT COUNT(*) FROM ticket` ≥ 20000 |
| ☐ | MS1-10 | Swagger completo + ejemplos + pruebas (happy path + validación de error) | 0.5d | MS1-03..08 | `/docs` completo; pruebas verdes |
| ☐ | MS1-11 | README (levantar local / tras corte) + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | Imagen publicada |
| ☐ | DS-15 | Carga masiva real: ejecutar `ingesta-ms1` **una vez** tras los 20k (con Mariano y Edinson) | 0.5d (repartido) | Backend F2 (20k) | S3 con el 100% de MS1 |
| ☐ | — | E/R de MySQL a `docs/er/ms1-mysql-er.png` / `.drawio` (actualizar tabla de [`docs/er/README.md`](../../er/README.md)) | — | MS1-02 | Diagrama entregado |
| ☐ | EX-05/06 | Poblar `docs/evidencias/backend/` + redactar tu sección del informe | 0.5d | verificación | Sección sin "TODO" |

---

## Entregas por hito

- **Hito 1:** MS1 con CRUD mínimo + MS1→MS2 + BD MySQL conectada; `ingesta-ms1` cargando a S3.
- **Hito 2:** MS1 completo + 20k en `ticket`/`equipaje` + Swagger + E/R MySQL.

## Informe

- **Sección 3 (Backend)** — tu microservicio: Swagger, `COUNT(*)` ≥ 20 000, log de consumo MS1→MS2.
- **Sección 3b** — E/R de MySQL.

## Dependencias clave

- **Necesitas de Mariano:** `MS2-07 /vuelos/{id}/exists` para MS1-05.
- **Necesitas de Benja:** `BE-TX-09` cliente HTTP compartido; `DS-04` bucket S3.
- **Necesitas de Fabricio:** `seeds/` con contrato de IDs para la carga 20k coherente.
- **Te esperan:** Fabricio (MS4 `/manifiesto` consume MS1-06).

## Nota de estado

⚠️ El repo **`ms1-pasajeros-api` todavía no existe** en la organización — hay que crearlo desde la plantilla.

## Con todo el equipo

- **BE-TX-01** — acordar `contratos/enums.md` y rangos de ID (F0).
- **EX-04** — exposición virtual con ACL (F1).
- **EX-11** — ensayo general de la demo el Vie 19 (F2).
- **EX-12** — exposición presencial + demo en vivo (F3, **obligatoria**).
