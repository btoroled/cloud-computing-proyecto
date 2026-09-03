# Plan de trabajo — Backend (Microservicios)

**Rúbrica:** 7 pts · **Hito 1:** avance ≥50% · **Hito 2:** completo.
Documentos base: [requerimientos §2–§5](../requerimientos.md) · [arquitectura](../arquitectura.md) ·
[enums](../contratos/enums.md) · [plan general](../plan-de-trabajo.md).

Convención de estimación: **d** = días-persona aproximados (jornada de estudiante ≈ 3–4 h).
Fases: **F0** = Setup/contratos (Mié 2 – Vie 4) · **F1** = Núcleo + deploy v1 (Sáb 6 – Sáb 12, Hito 1) ·
**F2** = Completar + 20k + endurecer (Sáb 13 – Vie 19).

---

## 1. Entregables y Definición de Terminado (DoD global)

- [ ] 5 microservicios en Docker: MS1 (Py/MySQL), MS2 (Java/PostgreSQL), MS3 (Node/MongoDB),
      MS4 (Py, sin BD), MS5 (Py, Athena).
- [ ] 3 lenguajes distintos + 3 BD distintas (2 SQL + 1 NoSQL); cada BD SQL con ≥2 tablas relacionadas.
- [ ] ≥1 microservicio consume a otro (MS1→MS2, MS3→MS2, MS4→MS1/MS2/MS3) — con evidencia en logs.
- [ ] 1 microservicio sin BD (MS4) + 1 microservicio analítico con Athena (MS5).
- [ ] Carga masiva **una vez** de ≥20 000 registros en ≥1 tabla por BD (`ticket`, `vuelo`, `incidencias`).
- [ ] Despliegue `docker compose` en **2 VMs de producción** + **ALB/NLB privado**; APIs públicas
      **solo** por **API Gateway HTTPS**; **3 BDs en VM privada** (sin IP pública).
- [ ] Swagger-UI navegable de las 5 APIs + página agregada.
- [ ] E/R de MySQL y PostgreSQL + JSON Schema de MongoDB en [`docs/er/`](../er/).
- [ ] Repos públicos con `README`, `openapi.yaml`, `Dockerfile`, CI a GHCR.

**DoD por microservicio:** endpoints del contrato implementados · `GET /health` OK · Swagger en `/docs` ·
`Dockerfile` + `compose` local · migraciones/seed reproducibles · imagen en GHCR por tag · pruebas
mínimas (happy path + validación de error) · README con "levantar local" y "levantar tras corte".

---

## 2. Equipo y asignación

| Componente | Responsable | Apoyo |
|---|---|---|
| Trabajo transversal (contratos, base Docker, CI, nginx, Swagger agregado) | Lead | Todos |
| MS1 — Pasajeros / Tickets | Dev A | — |
| MS2 — Vuelos / Operaciones | Dev B | — |
| MS3 — Infraestructura / Incidencias | Dev C | — |
| MS4 — Manifiesto de Vuelo | Dev D | — |
| MS5 — Analítico | Dev D | Dev E (visualización) |
| Integración, despliegue, red y seguridad | Lead | Dev B (tuning JVM) |

---

## 3. Trabajo transversal

| ID | Tarea | Resp. | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|---|
| BE-TX-01 | Acordar y versionar `contratos/enums.md` y rangos de ID | Lead + todos | 0.5d | F0 | — | Archivo aprobado en repo docs |
| BE-TX-02 | Plantilla de repo de microservicio (estructura, `.editorconfig`, `.env.example`, `README` base) | Lead | 0.5d | F0 | — | 5 repos creados desde plantilla |
| BE-TX-03 | `openapi.yaml` borrador por servicio (contract-first) | Cada dev | 0.5d c/u | F0 | BE-TX-01 | OpenAPI válido, revisado en standup Día 2 |
| BE-TX-04 | Imagen base Docker por lenguaje (Py 3.12-slim, Temurin 21, Node 20-alpine) + healthcheck | Lead | 0.5d | F0 | — | Imágenes construyen y publican a GHCR |
| BE-TX-05 | GitHub Actions: build + push a GHCR en tag `vX.Y` | Lead | 0.5d | F0/F1 | BE-TX-02 | Tag dispara imagen `ghcr.io/btoroled/<repo>:<tag>` |
| BE-TX-06 | `nginx.conf` reverse proxy por path (`/api/pasajeros`, `/api/vuelos`, …) | Lead | 0.5d | F1 | contratos | nginx enruta a los 5 servicios locales |
| BE-TX-07 | Página Swagger-UI agregada (lista los 5 `openapi.json`) | Dev E | 0.5d | F2 | BE-TX-03 | `/docs` agregado muestra los 5 |
| BE-TX-08 | Contrato de errores común (formato JSON de error, códigos 400/404/422/502) | Lead | 0.25d | F0 | — | Documentado en `contratos/` |
| BE-TX-09 | Cliente HTTP compartido para llamadas entre servicios (timeout, reintento, propagación de error) | Dev D | 0.5d | F1 | BE-TX-08 | Usado por MS1, MS3, MS4 |

---

## 4. MS1 — Pasajeros / Tickets · Python + FastAPI + MySQL 8 (Dev A)

| ID | Tarea | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|
| MS1-01 | Scaffold FastAPI + `Dockerfile` + `compose` local (app + mysql) | 0.5d | F0 | BE-TX-04 | `GET /health` responde en local |
| MS1-02 | Modelo y migraciones: `persona`, `pasajero`, `categoria_migratoria`, `ticket`, `checkin`, `equipaje` | 1d | F0/F1 | BE-TX-01 | Migración aplica; E/R exportado a `docs/er/ms1-mysql-er` |
| MS1-03 | CRUD `pasajeros` + búsqueda por clave alterna (`tipo_documento`+`numero_documento`) | 0.5d | F1 | MS1-02 | Endpoints §4.1 del requerimiento OK |
| MS1-04 | `GET /categorias-migratorias` + carga de las 3 categorías con tarifa TUUA | 0.25d | F1 | MS1-02 | Devuelve Nacional/Internacional/Transito |
| MS1-05 | `POST /tickets` con **validación de vuelo contra MS2** (`GET /vuelos/{id}/exists`) | 1d | F1 | MS2-07, BE-TX-09 | Vuelo inexistente/cancelado → 422; log muestra llamada saliente |
| MS1-06 | `GET /tickets/{id}`, `GET /tickets?vuelo_id=`, `GET /pasajeros/{id}/tickets` | 0.5d | F1 | MS1-05 | Filtros funcionan |
| MS1-07 | `POST /tickets/{id}/checkin` (1:1 con ticket) | 0.5d | F1 | MS1-05 | Segundo check-in del mismo ticket → 409 |
| MS1-08 | `POST /equipajes` + `GET /equipajes?pasajero_id=` / `?vuelo_id=` | 0.5d | F1 | MS1-02 | Tag ID único; peso válido |
| MS1-09 | Generador de datos ficticios + **carga masiva ≥20 000** en `ticket` y `equipaje` (una vez) | 1d | F2 | seeds compartido | `SELECT COUNT(*) FROM ticket` ≥ 20000 |
| MS1-10 | Swagger completo + ejemplos + pruebas (happy path + validación) | 0.5d | F2 | MS1-03..08 | `/docs` completo; pruebas verdes |
| MS1-11 | README (levantar local / tras corte) + tag `v1.0` → GHCR | 0.25d | F2 | BE-TX-05 | Imagen publicada |

---

## 5. MS2 — Vuelos / Operaciones · Java + Spring Boot + PostgreSQL 16 (Dev B)

| ID | Tarea | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|
| MS2-01 | Scaffold Spring Boot (web, data-jpa, actuator, springdoc) + `Dockerfile` + `compose` (app + postgres) | 0.5d | F0 | BE-TX-04 | `GET /health` (actuator) OK |
| MS2-02 | Entidades y migraciones (Flyway): `vuelo`, `aerolinea`, `aeronave`, `asiento`, `empleado`, `tripulacion`, `operativo_tierra`, `opera_tripulacion` | 1.5d | F0/F1 | BE-TX-01 | Migración aplica; E/R a `docs/er/ms2-postgres-er` |
| MS2-03 | CRUD `aerolineas` (PK natural `ruc`) + enum `alianza` | 0.5d | F1 | MS2-02 | Endpoints §4.2 OK |
| MS2-04 | CRUD `aeronaves` (PK natural `placa`) + `GET /aeronaves/{placa}/asientos` | 0.5d | F1 | MS2-02 | Lista de asientos por avión |
| MS2-05 | CRUD `vuelos` + filtros `?num=&estado=&tipo=&fecha=` | 1d | F1 | MS2-02 | Filtros combinables |
| MS2-06 | `PATCH /vuelos/{id}/estado` con máquina de estados (`Programado→…→Aterrizado`/`Cancelado`) | 0.5d | F1 | MS2-05 | Transición inválida → 422 |
| MS2-07 | `GET /vuelos/{id}/exists` (endpoint liviano para MS1/MS3) | 0.25d | F1 | MS2-05 | 200 `{exists,estado}` / 404 |
| MS2-08 | `empleados` + subclases `tripulacion`/`operativo_tierra`; `POST /vuelos/{id}/tripulacion`, `GET /vuelos/{id}/tripulacion` | 1d | F1/F2 | MS2-02 | Asignación N–M vía `opera_tripulacion` |
| MS2-09 | Generador + **carga masiva ≥20 000** en `vuelo` y `asiento` (una vez) | 1d | F2 | seeds compartido | `COUNT(*) vuelo` ≥ 20000, `asiento` ≥ 20000 |
| MS2-10 | Tuning JVM para `t3.small` (`-Xmx256m`, límite de memoria del contenedor) | 0.5d | F2 | MS2-01 | Contenedor estable < 400 MB RSS |
| MS2-11 | Swagger (springdoc) + pruebas (`@SpringBootTest` mínimas) | 0.5d | F2 | MS2-03..08 | `/docs` completo; pruebas verdes |
| MS2-12 | README + tag `v1.0` → GHCR | 0.25d | F2 | BE-TX-05 | Imagen publicada |

---

## 6. MS3 — Infraestructura / Incidencias · Node.js + Express + MongoDB 7 (Dev C)

| ID | Tarea | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|
| MS3-01 | Scaffold Express + `Dockerfile` + `compose` (app + mongo) + healthcheck | 0.5d | F0 | BE-TX-04 | `GET /health` OK |
| MS3-02 | JSON Schema de `recursos` (manga/radar), `incidencias`, `asignaciones` + validación con `ajv` | 1d | F0/F1 | BE-TX-01 | Schemas en `docs/er/ms3-mongo-schemas.md` |
| MS3-03 | CRUD `recursos` (manga\|radar) + `GET /recursos?tipo=&estado=` | 0.5d | F1 | MS3-02 | Filtro por estado/tipo |
| MS3-04 | `PATCH /recursos/{id}/estado` (Libre/Ocupado/Mantenimiento) | 0.25d | F1 | MS3-03 | Estado inválido → 422 |
| MS3-05 | `POST /incidencias` con **validación de vuelos contra MS2** + `GET /incidencias` + filtros `?tipo=&desde=&hasta=` | 1d | F1 | MS2-07, BE-TX-09 | `vuelo_id` inexistente → 422; log muestra llamada |
| MS3-06 | `GET /incidencias/{id}`, `PATCH /incidencias/{id}/cierre` (setea `fecha_cierre`) | 0.5d | F1 | MS3-05 | Incidencia cerrada no se re-cierra |
| MS3-07 | `POST /asignaciones` (Utiliza: vuelo↔recurso) + `GET /asignaciones?vuelo_id=` / `?recurso_id=` + regla clase manga ≥ clase aeronave | 0.75d | F1/F2 | MS3-03, MS2-07 | Acople de clase mayor a manga menor → 422 |
| MS3-08 | Generador + **carga masiva ≥20 000** en `incidencias` (una vez) | 1d | F2 | seeds compartido | `db.incidencias.countDocuments()` ≥ 20000 |
| MS3-09 | Swagger (`swagger-ui-express`) + pruebas (`jest` + `supertest`) | 0.5d | F2 | MS3-03..07 | `/docs` completo; pruebas verdes |
| MS3-10 | README + tag `v1.0` → GHCR | 0.25d | F2 | BE-TX-05 | Imagen publicada |

---

## 7. MS4 — Manifiesto de Vuelo · Python + FastAPI, sin BD (Dev D)

| ID | Tarea | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|
| MS4-01 | Scaffold FastAPI + `Dockerfile` (sin BD) + config de URLs de MS1/MS2/MS3 por entorno | 0.5d | F0 | BE-TX-04 | `GET /health/dependencias` reporta estado de los 3 |
| MS4-02 | `GET /manifiesto/{vuelo_id}` — agrega vuelo+aeronave+aerolínea (MS2), pasajeros+ticket+checkin+equipaje (MS1), tripulación (MS2), recursos+incidencias abiertas (MS3) | 1.5d | F1 | MS1-06, MS2-05/08, MS3-05/07, BE-TX-09 | Respuesta consolidada; vuelo inexistente → 404 |
| MS4-03 | `GET /manifiesto/{vuelo_id}/pasajeros` y `/resumen` (conteos: pax, kg equipaje, incidencias abiertas) | 0.5d | F2 | MS4-02 | Subrecursos coherentes con el manifiesto |
| MS4-04 | Manejo de dependencia caída (timeout / 502) → respuesta parcial con `warnings[]` | 0.5d | F2 | MS4-02 | Si MS3 cae, devuelve resto + warning |
| MS4-05 | Swagger + ejemplos + pruebas con mocks de MS1/MS2/MS3 | 0.5d | F2 | MS4-02 | `/docs` completo; pruebas verdes |
| MS4-06 | README + tag `v1.0` → GHCR | 0.25d | F2 | BE-TX-05 | Imagen publicada |

---

## 8. MS5 — Analítico · Python + FastAPI + boto3 → Athena (Dev D)

> **Depende de Data Science** (bucket S3 + catálogo Glue con datos). Ver [plan Data Science](data-science.md).

| ID | Tarea | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|
| MS5-01 | Scaffold FastAPI + `Dockerfile` + config Athena (workgroup, output S3, región) con `LabRole` | 0.5d | F1 | BE-TX-04 | `GET /health` OK; credenciales del lab cargadas |
| MS5-02 | Capa de ejecución Athena (lanzar query, *poll*, leer resultados, cachear por TTL) | 0.75d | F2 | DS catálogo | Ejecuta una query de prueba y devuelve filas |
| MS5-03 | `GET /analitica/recursos-mas-fallas?dias=` | 0.5d | F2 | MS5-02, DS Athena Q1 | Resultado con join recurso×incidencia |
| MS5-04 | `GET /analitica/retraso-promedio?tipo=` | 0.5d | F2 | MS5-02, DS Athena Q2 | Promedio en minutos por tipo de vuelo |
| MS5-05 | `GET /analitica/incidencias-combustible-por-aerolinea` | 0.5d | F2 | MS5-02, DS Athena Q3 | Ranking por aerolínea |
| MS5-06 | `GET /analitica/recaudacion-tuua-por-categoria` (lee `vw_recaudacion_tuua`) | 0.5d | F2 | MS5-02, DS vista 1 | Monto por categoría migratoria |
| MS5-07 | `GET /analitica/vuelos-hora-punta-retrasados` (lee `vw_retrasos_hora_punta`) | 0.5d | F2 | MS5-02, DS vista 2 | % de vuelos hora punta retrasados |
| MS5-08 | Swagger + pruebas (mock del cliente Athena) | 0.5d | F2 | MS5-03..07 | `/docs` completo; pruebas verdes |
| MS5-09 | README + tag `v1.0` → GHCR | 0.25d | F2 | BE-TX-05 | Imagen publicada |

---

## 9. Integración, despliegue, red y seguridad (Lead)

| ID | Tarea | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|
| BE-INT-01 | VPC + subredes privadas + IGW/NAT + Security Groups (§arquitectura §1) | 1d | F0 | Learner Lab OK | SGs mínimos aplicados |
| BE-INT-02 | 3 EC2: VM-PROD-1, VM-PROD-2, VM-DB + acceso por SSM | 0.5d | F0 | BE-INT-01 | Acceso sin puerto 22 público |
| BE-INT-03 | VM-DB: `compose` con `mysql:8` + `postgres:16` + `mongo:7` + volúmenes | 0.5d | F1 | BE-INT-02 | Los 3 motores `Up`; solo VM-PROD puede conectarse |
| BE-INT-04 | VM-PROD ×2: `compose` con nginx + MS1..MS5 (pull de GHCR) | 1d | F1 | BE-TX-06, imágenes | 5 servicios `Up` en ambas VMs |
| BE-INT-05 | ALB interno (o NLB) → target de las 2 VM-PROD:80 | 0.5d | F1 | BE-INT-04 | Balancea entre las 2 VMs |
| BE-INT-06 | API Gateway HTTP API + VPC Link → ALB; ruta `/{proxy+}` | 1d | F1 | BE-INT-05, Learner Lab | URL pública HTTPS responde `/api/*/health` |
| BE-INT-07 | `RUNBOOK.md` de reinicio tras corte de sesión (< 15 min) + script de dumps a S3 | 0.5d | F1/F2 | BE-INT-03 | Reinicio cronometrado documentado |
| BE-INT-08 | Prueba de humo E2E por la URL pública ([verificación](../verificacion.md)) + verificación de "privado" | 0.5d | F2 | todo F2 | Script E2E verde; ALB/BD inaccesibles desde fuera |
| BE-INT-09 | Logs de contenedores a CloudWatch (o recolección local) + healthchecks en `compose` | 0.5d | F2 | BE-INT-04 | Logs visibles; contenedores con healthcheck |

---

## 10. Mapa a hitos

**Hito 1 (Sáb 12-Set, ≥50% del Backend):**
- MS1, MS2, MS3 con CRUD mínimo y **≥1 BD conectada** (objetivo: las 3) y consultas básicas.
- MS1→MS2 y MS3→MS2 funcionando (consumo entre servicios).
- MS4 devolviendo un manifiesto simple.
- Deploy v1: 2 VM-PROD + ALB privado + API Gateway respondiendo (aunque falten endpoints).
- Seeds de prueba (~2–5k), **sin** exigir los 20 000 todavía.

**Hito 2 (Dom 20-Set, Backend completo):**
- Todos los endpoints de §4 del requerimiento.
- **20 000 registros** en `ticket`, `vuelo`, `incidencias`.
- MS5 con las 5 consultas Athena.
- Swagger-UI de las 5 + página agregada.
- E/R MySQL + PostgreSQL + JSON Schema Mongo entregados.
- Prueba de humo E2E y verificación de privacidad de ALB/BD.

---

## 11. Dependencias y camino crítico del Backend

```
contratos OpenAPI + enums (F0)
        │
        ├── MS2 modelo + /vuelos/{id}/exists ──┐
        │                                      ├── MS1 POST /tickets (consume MS2)
        │                                      ├── MS3 POST /incidencias (consume MS2)
        │                                      └── MS4 GET /manifiesto (consume MS1+MS2+MS3)
        │
infra red (F0) ── VM-DB (F1) ── VM-PROD compose (F1) ── ALB (F1) ── API Gateway+VPC Link (F1)
        │
seeds compartido ── carga 20k por BD (F2)
        │
Data Science (S3+Glue+Athena) ── MS5 (F2)
```

**Bloqueante principal:** `MS2-07 (/vuelos/{id}/exists)` habilita MS1-05, MS3-05 y MS4-02 → priorizar en
F1. **Segundo bloqueante:** API Gateway + VPC Link en Learner Lab (BE-INT-06) → validar viabilidad en F0.

---

## 12. Riesgos específicos del Backend

| Riesgo | Mitigación |
|---|---|
| API Gateway + VPC Link no disponible con `LabRole` | Validar en F0; contingencia NLB o exposición temporal del ALB solo para demo (ver [riesgos R2](../riesgos.md)) |
| 5 contenedores + JVM no caben en `t3.small` | `-Xmx256m` en MS2 (MS2-10); si no, VM-PROD a `t3.medium` (R7) |
| Seeds con IDs que no cruzan entre servicios | Orden MS2→MS1→MS3 y `SEED` fijo en `seeds/` compartido (R5) |
| MS5 bloqueado si Data Science se atrasa | MS5 se desarrolla en F2 contra el catálogo; sus tareas no están en el camino crítico de Hito 1 |
| Corte de sesión borra datos de la VM-DB | Dumps a S3 y `RUNBOOK.md` (BE-INT-07) |
