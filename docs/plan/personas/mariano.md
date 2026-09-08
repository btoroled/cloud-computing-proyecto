# Mariano — MS2 Vuelos / Operaciones ⚠️

**Repo:** `ms2-vuelos-api` · **Stack:** Java + Spring Boot + PostgreSQL 16 · **Ingesta:** `ingesta-ms2`
**Carga estimada:** ~12 d — en el límite de capacidad.

Fuentes: [backend.md](../backend.md) §5 · [data-science.md](../data-science.md) §5 ·
[exposiciones-e-informe.md](../exposiciones-e-informe.md) · [enums](../../contratos/enums.md).

> **Eres el bloqueante principal del Backend:** `MS2-07 /vuelos/{id}/exists` habilita MS1-05, MS3-05 y
> MS4-02. Priorízalo en F1.

---

## F0 — Setup y contratos (Mié 2 – Vie 4)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-TX-03 | `openapi.yaml` borrador de MS2 (contract-first) | 0.5d | BE-TX-01 | OpenAPI válido, revisado en standup Día 2 |
| ☐ | MS2-01 | Scaffold Spring Boot (web, data-jpa, actuator, springdoc) + `Dockerfile` + `compose` (app + postgres) | 0.5d | BE-TX-04 | `GET /health` (actuator) OK |
| ☐ | MS2-02 | Entidades y migraciones (Flyway): `vuelo`, `aerolinea`, `aeronave`, `asiento`, `empleado`, `tripulacion`, `operativo_tierra`, `opera_tripulacion` | 1.5d | BE-TX-01 | Migración aplica; E/R a `docs/er/ms2-postgres-er` |
| ☐ | — | `docker compose up` local del servicio + su BD, con migraciones y 1 endpoint real | — | MS2-01/02 | Servicio arranca con datos |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS2-03 | CRUD `aerolineas` (PK natural `ruc`) + enum `alianza` | 0.5d | MS2-02 | Endpoints §4.2 OK |
| ☐ | MS2-04 | CRUD `aeronaves` (PK natural `placa`) + `GET /aeronaves/{placa}/asientos` | 0.5d | MS2-02 | Lista de asientos por avión |
| ☐ | MS2-05 | CRUD `vuelos` + filtros `?num=&estado=&tipo=&fecha=` | 1d | MS2-02 | Filtros combinables |
| ☐ | MS2-06 | `PATCH /vuelos/{id}/estado` con máquina de estados (`Programado→…→Aterrizado`/`Cancelado`) | 0.5d | MS2-05 | Transición inválida → 422 |
| ☐ | MS2-07 | `GET /vuelos/{id}/exists` (endpoint liviano para MS1/MS3) — **PRIORIDAD** | 0.25d | MS2-05 | 200 `{exists,estado}` / 404 |
| ☐ | MS2-08 | `empleados` + subclases `tripulacion`/`operativo_tierra`; `POST`/`GET /vuelos/{id}/tripulacion` | 1d | MS2-02 | Asignación N–M vía `opera_tripulacion` |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | MS2-09 | Generador + **carga masiva ≥20 000** en `vuelo` y `asiento` (una vez) | 1d | seeds compartido | `COUNT(*) vuelo` ≥ 20000, `asiento` ≥ 20000 |
| ☐ | MS2-10 | Tuning JVM para `t3.small` (`-Xmx256m`, límite de memoria del contenedor) | 0.5d | MS2-01 | Contenedor estable < 400 MB RSS |
| ☐ | MS2-11 | Swagger (springdoc) + pruebas (`@SpringBootTest` mínimas) | 0.5d | MS2-03..08 | `/docs` completo; pruebas verdes |
| ☐ | MS2-12 | README + tag `v1.0` → GHCR | 0.25d | BE-TX-05 | Imagen publicada |
| ☐ | DS-07 | **`ingesta-ms2`**: PostgreSQL → CSV → `raw/ms2/` (8 tablas) | 1d | DS-04, MS2-02 | Archivos de las 8 tablas en S3 |
| ☐ | DS-15 | Carga masiva real: ejecutar `ingesta-ms2` **una vez** tras los 20k (con Guillermo y Edinson) | 0.5d (repartido) | Backend F2 (20k) | S3 con el 100% de MS2 |
| ☐ | — | E/R de PostgreSQL a `docs/er/ms2-postgres-er.png` / `.drawio` | — | MS2-02 | Diagrama entregado |
| ☐ | EX-05/06 | Poblar `docs/evidencias/backend/` + redactar tu sección del informe | 0.5d | verificación | Sección sin "TODO" |

---

## Entregas por hito

- **Hito 1:** MS2 con CRUD mínimo + `/vuelos/{id}/exists` + BD PostgreSQL conectada.
- **Hito 2:** MS2 completo + 20k en `vuelo`/`asiento` + Swagger + E/R PostgreSQL + `ingesta-ms2`.

## Informe

- **Sección 3 (Backend)** — tu microservicio: Swagger, `COUNT(*)` ≥ 20 000 en `vuelo`.
- **Sección 3b** — E/R de PostgreSQL.

## Apoyo que das

- **Tuning JVM** en el despliegue (con Benja, tarea de integración).
- Si Edinson va adelantado, **te ayuda** con el generador de `asiento` / `opera_tripulacion`.

## Riesgos a tu cargo

- **R7** (con Benja) Java/Spring Boot pesado en `t3.small` → `-Xmx256m` + límite de memoria; si compite por RAM, VM-PROD a `t3.medium`.

## Nota de estado

⚠️ El repo **`ms2-vuelos-api` todavía no existe** en la organización — hay que crearlo desde la plantilla.

## Con todo el equipo

- **BE-TX-01** — acordar `contratos/enums.md` y rangos de ID (F0).
- **EX-04** — exposición virtual con ACL (F1).
- **EX-11** — ensayo general de la demo el Vie 19 (F2).
- **EX-12** — exposición presencial + demo en vivo (F3, **obligatoria**).
