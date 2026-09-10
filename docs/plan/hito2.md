# Hito 2 — Entrega completa (Dom 20-Set 23:59, Canvas) + exposición presencial (Semana 7)

Todo lo que **falta** tras el Hito 1. Fase **F2** = completar + cargar 20k + endurecer (Sáb 13 – Vie 19),
**F3** = entrega + exposición (Sáb 20 / Semana 7).

Documentos base: [plan general §6](../plan-de-trabajo.md#6-checklist-hito-2--entregables-finales) ·
[backend.md](backend.md) · [frontend.md](frontend.md) · [data-science.md](data-science.md) ·
[diagrama-arquitectura.md](diagrama-arquitectura.md) · [exposiciones-e-informe.md](exposiciones-e-informe.md) ·
[verificación](../verificacion.md).

---

## 1. Definición de Terminado global (criterios del enunciado)

- [ ] **5 microservicios** en Docker, **3 lenguajes** distintos + **3 BD** distintas (2 SQL + 1 NoSQL)
- [ ] Cada BD SQL con **≥2 tablas relacionadas**; **E/R** de MySQL y PostgreSQL + **JSON Schema** de MongoDB
- [ ] **≥1 microservicio consume a otro** (MS1→MS2, MS3→MS2, MS4→MS1/MS2/MS3) — con evidencia en logs
- [ ] **1 microservicio sin BD** (MS4) + **1 microservicio analítico con Athena** (MS5)
- [ ] **Carga masiva ≥20 000 registros**, una vez, en ≥1 tabla por BD (`ticket`, `vuelo`, `incidencias`) — evidencia `COUNT(*)`
- [ ] Despliegue `docker compose` en **2 VM-PROD** + **ALB/NLB privado**; APIs públicas **solo** por **API Gateway HTTPS**; **3 BD en VM privada** sin IP pública
- [ ] **Swagger-UI** navegable de las 5 APIs + página agregada
- [ ] **Frontend en Amplify** con **4 vistas** consumiendo **los 5 microservicios**, **≥2 métodos REST c/u**
- [ ] **Data Science:** bucket S3 + **3 contenedores de ingesta (pull 100%)** + **catálogo Glue por archivo** + **E/R del catálogo** + **≥4 consultas Athena con join** + **≥2 vistas**
- [ ] **Diagrama de arquitectura de solución** (`draw.io`) con todos los servicios AWS
- [ ] **Informe Word/PDF** + **PPT** con evidencias · `INDEX.md` con enlaces a los repos públicos
- [ ] Exposición **presencial + demo en vivo** (Semana 7)

---

## 2. Backend — lo que falta

### 2.1 Transversal (Benja / Fabricio)

| ID | Tarea | Resp. | DoD |
|---|---|---|---|
| BE-TX-03 | `openapi.yaml` definitivo por servicio (contract-first) | Cada dev | OpenAPI válido de los 5 |
| BE-TX-06 | `nginx.conf` reverse proxy por path (`/api/pasajeros`, `/api/vuelos`, `/api/infra`, `/api/manifiesto`, `/api/analitica`) | Benja | nginx enruta a los 5 servicios |
| BE-TX-07 | Página **Swagger-UI agregada** (lista los 5 `openapi.json`) | Alexander | `/docs` agregado muestra los 5 |
| BE-TX-09 | Cliente HTTP compartido entre servicios (timeout, reintento, propagación de error) | Fabricio | Usado por MS1, MS3, MS4 |

### 2.2 MS1 — Pasajeros / Tickets · Python + FastAPI + MySQL (Guillermo)

Todo MS1 está pendiente (hoy es scaffold vacío): MS1-01..MS1-11 de [backend §4](backend.md#4-ms1--pasajeros--tickets--python--fastapi--mysql-8-guillermo).
Prioridad: scaffold + `Dockerfile` + `compose` → modelo/migraciones (`persona`, `pasajero`,
`categoria_migratoria`, `ticket`, `checkin`, `equipaje`) → CRUD pasajeros → `POST /tickets` con
**validación de vuelo contra MS2** → check-in / equipajes → **seed ≥20 000** en `ticket` → Swagger +
pruebas → README + tag `v1.0` a GHCR.

### 2.3 MS2 — Vuelos / Operaciones · Java + Spring Boot + PostgreSQL (Mariano)

Todo MS2 pendiente: MS2-01..MS2-12 de [backend §5](backend.md#5-ms2--vuelos--operaciones--java--spring-boot--postgresql-16-mariano).
Prioridad: scaffold Spring Boot (web, data-jpa, actuator, springdoc) → entidades + Flyway (`vuelo`,
`aerolinea`, `aeronave`, `asiento`, `empleado`, `tripulacion`, `operativo_tierra`, `opera_tripulacion`)
→ CRUD aerolíneas/aeronaves/vuelos + filtros → `PATCH /vuelos/{id}/estado` (máquina de estados) →
**`GET /vuelos/{id}/exists`** (bloqueante de MS1/MS3/MS4 — priorizar) → tripulación N–M → **seed
≥20 000** en `vuelo` y `asiento` → **tuning JVM** (`-Xmx256m`) para `t3.small` → Swagger + pruebas →
README + tag.

### 2.4 MS3 — Infraestructura / Incidencias · Node + Express + MongoDB (Edinson)

Hecho en Hito 1: `recursos` (GET lista, GET id, POST). Falta:

| ID | Tarea | DoD |
|---|---|---|
| MS3-02 | JSON Schema de `recursos`, `incidencias`, `asignaciones` documentado en `docs/er/ms3-mongo-schemas.md` | Schemas versionados |
| MS3-04 | `PATCH /recursos/{id}/estado` (Libre/Ocupado/Mantenimiento) si no se hizo en Hito 1 | Estado inválido → 422 |
| MS3-05 | `POST /incidencias` con **validación de vuelos contra MS2** + `GET /incidencias` + filtros `?tipo=&desde=&hasta=` | `vuelo_id` inexistente → 422; log muestra la llamada |
| MS3-06 | `GET /incidencias/{id}`, `PATCH /incidencias/{id}/cierre` | Incidencia cerrada no se re-cierra |
| MS3-07 | `POST /asignaciones` (vuelo↔recurso) + `GET /asignaciones?vuelo_id=`/`?recurso_id=` + regla clase manga ≥ clase aeronave | Acople inválido → 422 |
| MS3-08 | Generador + **carga masiva ≥20 000** en `incidencias` (una vez) | `db.incidencias.countDocuments()` ≥ 20000 |
| MS3-09 | Swagger (`swagger-ui-express`) + pruebas (`jest` + `supertest`) | `/docs` completo; pruebas verdes |
| MS3-10 | README (levantar local / tras corte) + tag `v1.0` → GHCR | Imagen publicada |

### 2.5 MS4 — Manifiesto de Vuelo · Python + FastAPI, sin BD (Fabricio)

Todo MS4 pendiente: MS4-01..MS4-06 de [backend §7](backend.md#7-ms4--manifiesto-de-vuelo--python--fastapi-sin-bd-fabricio).
`GET /manifiesto/{vuelo_id}` agrega MS1 + MS2 + MS3 → `/pasajeros` y `/resumen` → manejo de dependencia
caída (respuesta parcial con `warnings[]`) → Swagger + pruebas con mocks → README + tag.

### 2.6 MS5 — Analítico · Python + FastAPI + boto3 → Athena (Fabricio)

Todo MS5 pendiente: MS5-01..MS5-09 de [backend §8](backend.md#8-ms5--analítico--python--fastapi--boto3--athena-fabricio).
Depende del catálogo Glue + Athena (Data Science). Scaffold + config Athena (`LabRole`) → capa de
ejecución Athena (poll + cache TTL) → 5 endpoints `GET /analitica/*` (Q1–Q5) → Swagger + pruebas
(mock del cliente Athena) → README + tag.

### 2.7 Integración, red y seguridad (Benja)

| ID | Tarea | DoD |
|---|---|---|
| BE-INT-03 | VM-DB: `compose` con los 3 motores + volúmenes; solo VM-PROD/VM-INGESTA pueden conectarse | 3 motores `Up`, acceso restringido por SG |
| BE-INT-04 | **VM-PROD ×2**: `compose` con nginx + MS1..MS5 (pull de GHCR) | 5 servicios `Up` en ambas VMs |
| BE-INT-05 | **ALB interno** (o NLB) → target de las 2 VM-PROD:80 | Balancea entre las 2 VMs; health checks `healthy` |
| BE-INT-06 | **API Gateway HTTP API + VPC Link → ALB**; ruta `/{proxy+}`; **cerrar `sg-vm-prod`** (quitar el 0.0.0.0/0 del Hito 1, dejar solo desde `sg-alb`) | URL HTTPS responde `/api/*/health`; VM-PROD ya no es pública |
| BE-INT-07 | `RUNBOOK.md` de reinicio < 15 min + script de dumps a S3 (ya existe — validar cronometrado) | Reinicio real cronometrado y documentado |
| BE-INT-08 | Prueba de humo **E2E** por la URL pública ([verificación](../verificacion.md)) + verificación de "privado" (`nc -zv <alb-dns> 80` **falla** desde fuera; VM-DB sin IP pública) | Script E2E verde; ALB/BD inaccesibles desde fuera |
| BE-INT-09 | Logs de contenedores a CloudWatch (o recolección local) + healthchecks en `compose` | Logs visibles; contenedores con healthcheck |

### 2.8 Consumo entre microservicios (evidencia obligatoria)

- [ ] **MS1 → MS2** en `POST /tickets` (valida vuelo con `GET /vuelos/{id}/exists`) — log de la llamada saliente
- [ ] **MS3 → MS2** en `POST /incidencias` — log de la llamada saliente
- [ ] **MS4 → MS1/MS2/MS3** en `GET /manifiesto/{vuelo_id}` — respuesta consolidada

---

## 3. Frontend — lo que falta (Alexander)

Hecho en Hito 1: Amplify + 1 vista consumiendo MS3. Falta llevarlo a **4 vistas / 5 MS / ≥2 métodos c/u**.

| ID | Tarea | DoD |
|---|---|---|
| FE-06 | **Vista "Consulta de vuelo + manifiesto"** — `GET /vuelos`, `GET /vuelos/{id}` (MS2) + `GET /manifiesto/{id}`, `/resumen` (MS4) | Busca vuelo y muestra manifiesto consolidado |
| FE-08 | **Vista "Emisión de ticket / check-in"** — `GET /categorias-migratorias`, `POST /tickets`, `POST /tickets/{id}/checkin` (MS1) + `GET /vuelos` (MS2) | Emite ticket y hace check-in end-to-end |
| FE-09 | **Vista "Recursos e incidencias"** — completar con `POST /incidencias`, `GET /incidencias`, `PATCH /recursos/{id}/estado` (MS3) | Lista recursos libres y crea incidencia |
| FE-10 | **Vista "Dashboard de crisis"** — 5 indicadores/gráficos desde MS5 (Alexander + Fabricio) | 5 tarjetas/gráficos con datos reales de Athena |
| FE-11 | Página **Swagger-UI agregada** (selector de los 5 `openapi.json`) | `/docs` agregado navegable |
| FE-12 | Pulido: responsive, estados vacíos, manejo de dependencia caída | Sin errores de consola; funciona en móvil |
| FE-13 | README + capturas para el informe (matriz de cobertura REST del [frontend §3](frontend.md#3-matriz-de-cobertura-rest-requisito-2-métodos-por-microservicio)) | Sección de informe lista |

Requisito a cerrar: **matriz de cobertura REST** — cada uno de los 5 MS invocado con ≥2 métodos
distintos, documentado con capturas del panel Network.

---

## 4. Data Science — lo que falta (Fabricio + devs de BD)

Hecho en Hito 1: bucket S3 + VM-INGESTA + `ingesta-ms3` con datos en S3. Falta:

| ID | Tarea | Resp. | DoD |
|---|---|---|---|
| DS-02 | `seeds/` v1 completo: `SEED` fijo, orden **MS2→MS1→MS3**, rangos de ID de [plan general §3](../plan-de-trabajo.md#3-datos-de-prueba-integridad-entre-servicios) | Fabricio | CSV coherentes entre los 3 dominios |
| DS-03 | Las 5 consultas (Q1–Q5) en **SQL sobre Postgres local** para validar la lógica | Fabricio | Las 5 devuelven resultados razonables |
| DS-06 | **`ingesta-ms1`** (MySQL → CSV → `raw/ms1/`, 6 tablas, pull 100%) | Guillermo | Archivos de las 6 tablas en S3 |
| DS-07 | **`ingesta-ms2`** (PostgreSQL → CSV → `raw/ms2/`, 8 tablas, pull 100%) | Mariano | Archivos de las 8 tablas en S3 |
| DS-08 | **`ingesta-ms3`** completa: leer colecciones + **aplanar `incidencias`** en `incidencia` / `incidencia_recurso` / `incidencia_vuelo` → `raw/ms3/` | Edinson | 5 archivos en S3; conteos cuadran |
| DS-09 | Glue: database `aeropuerto_lake` + **1 crawler por prefijo** `raw/msX/` | Fabricio | Tablas visibles en el catálogo |
| DS-10 | Ajuste de esquemas inferidos por el crawler (tipos, `timestamp`, columnas) | Fabricio | `SELECT * LIMIT 10` correcto por tabla |
| DS-11 | Implementar **Q1–Q5 en Athena** (workgroup + output S3) — ver [data-science §4](data-science.md#4-consultas-athena-las-5-que-respaldan-ms5) | Fabricio | Las 5 corren, hacen JOIN y devuelven filas |
| DS-12 | Crear las **2 vistas** `vw_recaudacion_tuua` y `vw_retrasos_hora_punta` | Fabricio | `SHOW VIEWS` las lista; DDL en `athena/` |
| DS-13 | **E/R del catálogo** (`diagramas/er-catalogo-datalake.drawio`) con todas las tablas + claves de join | Fabricio / Alexander | Diagrama entregado |
| DS-14 | Evidencias en `docs/evidencias/athena/` (5 queries + 2 vistas + `aws s3 ls` + Glue console) | Fabricio | Carpeta completa |
| DS-15 | **Carga masiva real:** ejecutar la ingesta **una vez** tras los 20 000 registros del Backend | Guillermo/Mariano/Edinson | S3 con el 100% de los datos de las 3 BD |

---

## 5. Diagrama de arquitectura de solución (Benja) · 1 pt

Ya se entregó la **v1 (arquitectura planificada)** en el Hito 1 ([hito1 §3.6](hito1.md#36-track-e--diagrama-de-arquitectura-en-drawio-benja)):
`diagramas/arquitectura-solucion.drawio` + `.png`. En Hito 2 se **actualiza a la infra final**.

| Tarea | DoD |
|---|---|
| Actualizar el `.drawio` para que refleje **todo** lo desplegado: VPC/subredes/SG, NAT/IGW, endpoint S3, 4 EC2 (VM-PROD ×2, VM-DB, VM-INGESTA), **ALB interno**, **API Gateway + VPC Link**, S3, Glue, Athena, Amplify, los 5 microservicios; flujos de request y de datos | Diagrama en `diagramas/`, PNG re-exportado, referenciado en el informe |
| Quitar el marcado gris/punteado de "pendiente Hito 2" que traía la v1; versión final **coincide con la infra real desplegada** y con el boceto Mermaid `diagramas/arquitectura-solucion.md` | Diagrama revisado contra la consola AWS |

---

## 6. Documentación de datos (E/R + JSON Schema)

- [ ] **E/R MySQL** (MS1) en `docs/er/ms1-mysql-er.*` — ≥2 tablas relacionadas
- [ ] **E/R PostgreSQL** (MS2) en `docs/er/ms2-postgres-er.*` — ≥2 tablas relacionadas
- [ ] **JSON Schema MongoDB** (MS3) en `docs/er/ms3-mongo-schemas.md`
- [ ] **E/R del catálogo del data lake** (DS-13)

---

## 7. Exposición e informe (Benja consolida) · 1 + 3 pts

Ver [exposiciones-e-informe.md](exposiciones-e-informe.md).

| Entregable | DoD |
|---|---|
| **Exposición virtual ACL** (Hito 1, ya pasada) | Registrada |
| **Informe Word/PDF** — usar `informe/plantilla-informe.md`; una sección por componente con evidencias | PDF en Canvas |
| **PPT resumen** | Subido a Canvas |
| **`INDEX.md`** con enlaces a los 9 repos públicos | Todos los links funcionan |
| **Guion de demo** (`informe/guion-demo.md`) + ensayo el Vie 19 | Demo de ≤10 min cronometrada |
| **Exposición presencial + demo en vivo** (Semana 7) | Todos los integrantes participan |

---

## 8. Camino crítico del Hito 2

```
seeds/ (DS-02) ──► 20k en las 3 BD (MS1-09, MS2-09, MS3-08) ──► ingesta pull 100% (DS-06..08, DS-15)
                                                                        │
                                                                        ▼
                                    S3 ──► Glue crawlers (DS-09/10) ──► Athena Q1–Q5 + vistas (DS-11/12)
                                                                                    │
                                                                                    ▼
                                                                    MS5 (§2.6) ──► Dashboard Frontend (FE-10)

infra red ──► VM-DB (BE-INT-03) ──► VM-PROD ×2 (BE-INT-04) ──► ALB (BE-INT-05) ──► API Gateway + VPC Link (BE-INT-06) ──► E2E (BE-INT-08)
```

**Bloqueante #1:** `MS2-07 (/vuelos/{id}/exists)` habilita MS1-05, MS3-05 y MS4-02 → primero en F2.
**Bloqueante #2:** la cadena `seeds → 20k → ingesta → Glue → Athena → MS5 → Dashboard` es la más larga
del proyecto; empezar `seeds/` y `ingesta-ms1` cuanto antes.
**Bloqueante #3:** API Gateway + VPC Link + ALB privado en Learner Lab (BE-INT-06).

---

## 9. Sugerencia de orden F2 (Sáb 13 – Vie 19)

| Día | Foco |
|---|---|
| Sáb 13 | MS2 scaffold + modelo + `/vuelos/{id}/exists` (Mariano) · MS1 scaffold + modelo (Guillermo) · `seeds/` v1 (Fabricio) · ALB + cerrar SGs (Benja) |
| Dom 14 | MS1 CRUD + `POST /tickets`→MS2 · MS3 incidencias + asignaciones · API Gateway + VPC Link privado (Benja) |
| Lun 15 | MS4 manifiesto (consume MS1/2/3) · Frontend FE-06 + FE-08 · `ingesta-ms1`/`ingesta-ms2` |
| Mar 16 | Carga **20 000** en las 3 BD · `ingesta-ms3` con aplanado · Glue crawlers |
| Mié 17 | Athena Q1–Q5 + 2 vistas · MS5 endpoints · Frontend FE-09 |
| Jue 18 | Dashboard FE-10 (MS5) · Swagger agregado (FE-11) · E/R MySQL/Postgres/Mongo · diagrama arquitectura |
| Vie 19 | E2E (BE-INT-08) · verificación privacidad · pulido FE-12 · **code freeze** · consolidar informe + PPT · ensayo demo |
| Sáb 20 | Buffer + subir a Canvas (informe + PPT + `INDEX.md`) |

---

## 10. Riesgos abiertos

| Riesgo | Mitigación |
|---|---|
| MS1/MS2/MS4/MS5 arrancan de cero en F2 (4 servicios en 7 días) | Priorizar el camino crítico (MS2-07 → MS1/MS3 → MS4); MS5 va contra mocks hasta que Athena responda |
| API Gateway + VPC Link no disponible con `LabRole` | Contingencia NLB o ALB público solo para demo ([riesgos R2](../riesgos.md)) |
| 5 contenedores + JVM no caben en `t3.small` | `-Xmx256m` en MS2 (MS2-10); si no, VM-PROD a `t3.medium` |
| Seeds con IDs que no cruzan → joins Athena vacíos | Orden MS2→MS1→MS3 + `SEED` fijo; DS-03 valida joins en local |
| Corte de sesión borra datos de VM-DB | Dumps a S3 + RUNBOOK Parte 2 |
| Carga de Alexander alta (~13 d de frontend) | Fabricio toma FE-10; recortar FE-12 al mínimo si aprieta |
| URL de API Gateway cambia tras recrear infra | El frontend la lee de `VITE_API_BASE` en build; rebuild de Amplify tras cada recreación |
