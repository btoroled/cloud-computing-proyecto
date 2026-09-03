# Plan de Trabajo

Proyecto Parcial CS2032 — Cloud Computing (Ciclo 2026-2).

Documentos relacionados: [requerimientos](requerimientos.md) · [arquitectura](arquitectura.md) ·
[riesgos](riesgos.md) · [verificación](verificacion.md) · [índice de repos](../INDEX.md).

**Equipo:** 6 integrantes — 1 lead dev / arquitecto de solución (Benjamín) + 5 devs del mismo nivel.
**Entorno AWS:** AWS Academy Learner Lab.
**Fechas (2026):** hoy Mié 2-Set · **Hito 1 (ACL)** Sáb 12-Set 23:59 · **Hito 2 (Canvas)** Dom 20-Set
23:59 · **Exposición presencial** Semana 7.

---

## 1. Estrategia de repositorios

| Repo | Contenido | Dueño |
|---|---|---|
| `cloud-computing-proyecto` (este) | Toda la documentación: `docs/`, `diagramas/`, `informe/`, `ppt/`, `INDEX.md` | Lead |
| `ms1-pasajeros-api` | MS1 Python/FastAPI/MySQL + `Dockerfile` + `compose` local + migraciones + `openapi.yaml` + tests | Dev A |
| `ms2-vuelos-api` | MS2 Java/Spring Boot/PostgreSQL + ídem | Dev B |
| `ms3-infraestructura-api` | MS3 Node/Express/MongoDB + ídem | Dev C |
| `ms4-manifiesto-api` | MS4 Python/FastAPI (sin BD) + ídem | Dev D |
| `ms5-analitica-api` | MS5 Python/FastAPI + boto3 + SQL de las consultas Athena | Dev D |
| `aeropuerto-frontend` | SPA React + config Amplify + página Swagger-UI agregada | Dev E |
| `aeropuerto-data-science` | 3 contenedores de ingesta + `seeds/` (generador de data ficticia) + scripts Glue + DDL de vistas Athena | Dev D + A/B/C |
| `aeropuerto-infra-deploy` | Terraform/scripts de VPC+EC2+SG, `compose` de VM-PROD y VM-DB, `nginx.conf`, config API Gateway + VPC Link, `RUNBOOK.md` | Lead |

Convenciones: rama `main` protegida, trabajo por `feature/*`, PR con 1 revisión. Tag `vX.Y` dispara
build de imagen a GHCR. `LICENSE` MIT + `README` en todos.

---

## 2. Organización del equipo (6 personas)

| Rol | Persona | Responsabilidades |
|---|---|---|
| **Lead / Arquitecto** | Benjamín | `aeropuerto-infra-deploy` completo · VPC/subredes/SG · 4 EC2 · ALB interno · API Gateway + VPC Link · nginx gateway · despliegue `compose` en las 3 VMs · diagrama `draw.io` de arquitectura · pruebas de integración E2E · control del camino crítico · consolidación del informe |
| **Dev A** | — | `ms1-pasajeros-api` · E/R MySQL · llamada MS1→MS2 · contenedor `ingesta-ms1` · seed 20k `ticket`/`equipaje` |
| **Dev B** | — | `ms2-vuelos-api` · E/R PostgreSQL · endpoint `/vuelos/{id}/exists` · contenedor `ingesta-ms2` · seed 20k `vuelo`/`asiento` |
| **Dev C** | — | `ms3-infraestructura-api` · JSON schemas · llamada MS3→MS2 · contenedor `ingesta-ms3` (con aplanado) · seed 20k `incidencias` |
| **Dev D** | — | `ms4-manifiesto-api` · `ms5-analitica-api` · catálogo Glue · ≥4 consultas Athena + ≥2 vistas · E/R del data lake · `seeds/` compartido |
| **Dev E** | — | `aeropuerto-frontend` (4 vistas) · deploy Amplify + contingencia · página Swagger-UI agregada · apoyo a Dev D en gráficos del dashboard |

Ceremonias: **standup diario de 15 min** con el Lead (bloqueos + camino crítico); revisión de contratos
OpenAPI el Día 2; ensayo de demo el Vie 19.

---

## 3. Datos de prueba: integridad entre servicios

La data ficticia se genera en orden **MS2 → MS1 → MS3** para que las referencias suaves apunten a
registros existentes. Script semilla compartido en `aeropuerto-data-science/seeds/` con un `SEED` fijo
(reproducible).

| Entidad | Rango de ID |
|---|---|
| `vuelo.id` | 1 – 25 000 |
| `aerolinea.ruc` | RUCs generados (11 dígitos) |
| `aeronave.placa` | `OB-####` |
| `persona.id` / `pasajero.id` | 100 000 – 160 000 |
| `ticket.id` | 1 – 60 000 |
| `recurso.id` | 1 – 500 |
| `incidencia.id` | 1 – 30 000 (cada una referida a `vuelo_id` ∈ [1, 25 000] y/o `recurso.id` ∈ [1, 500]) |

Tablas objetivo para el volumen ≥20 000: `ticket` / `equipaje` (MySQL), `vuelo` / `asiento`
(PostgreSQL), `incidencias` (MongoDB).

---

## 4. Cronograma (Mié 2-Set → Dom 20-Set; presencial Semana 7)

### Fase 0 — Setup y contratos · Mié 2 – Vie 4
- **Lead:** crear org GitHub + los 9 repos con plantilla; invitar al equipo. Iniciar Learner Lab y
  **validar en orden**: EC2+VPC → S3 → Glue+Athena → API Gateway+VPC Link → Amplify. Registrar qué
  está disponible en [`docs/aws-learner-lab-hallazgos.md`](aws-learner-lab-hallazgos.md). Crear VPC,
  subredes, SG, NAT/IGW y 4 EC2 vacías.
- **Todos:** publicar `openapi.yaml` borrador del propio servicio; acordar
  [`docs/contratos/enums.md`](contratos/enums.md) y los rangos de ID (§3).
- **A/B/C:** `docker compose up` local del servicio + su BD, con migraciones y 1 endpoint real.
- **D:** estructura de `aeropuerto-data-science`; escribir las 5 consultas de negocio en SQL sobre el
  modelo relacional (validarlas en Postgres local antes de Athena).
- **E:** scaffold React desplegado (Amplify o contingencia) con 1 llamada real a MS2 (`GET /vuelos`).
- **Salida V0 (Vie 4):** cada servicio responde `/health` + 1 endpoint; frontend desplegado; hallazgos
  del lab documentados.

### Fase 1 — Núcleo funcional + despliegue v1 · Sáb 6 – Sáb 12 → **Hito 1**
- **A:** CRUD de pasajeros/tickets/checkin/equipaje; llamada MS1→MS2 al emitir ticket; seed ~5k.
- **B:** CRUD de vuelos/aerolíneas/aeronaves/asientos/tripulación; `/vuelos/{id}/exists`; seed ~2k vuelos.
- **C:** CRUD de recursos/incidencias/asignaciones; llamada MS3→MS2; seed ~5k incidencias.
- **D:** `GET /manifiesto/{vuelo_id}` consumiendo A/B/C; **VM-INGESTA + bucket S3 + contenedor
  `ingesta-ms1`** cargando datos reales a S3.
- **E:** frontend desplegado consumiendo **≥1 microservicio con ≥2 métodos REST**.
- **Lead:** nginx gateway en las 2 VM-PROD; `compose` con los 5 servicios; ALB interno; API Gateway +
  VPC Link; SG cerrados; runbook de reinicio.
- **Sáb 12-Set 23:59 — Hito 1** (exposición virtual ACL, 3 pts). Checklist en §5.

### Fase 2 — Completar, cargar 20k, endurecer · Sáb 13 – Vie 19
- **A/B/C:** completar endpoints; **carga masiva ≥20 000 registros por BD (una vez)**; validación de
  integridad cruzada; Swagger completo.
- **C:** aplanado de `incidencias` en el contenedor de ingesta.
- **D:** los **3 contenedores de ingesta** corriendo (pull 100%) → S3; **crawlers Glue** → catálogo;
  **≥4 consultas Athena con join + ≥2 vistas**; MS5 exponiendo las 5 consultas; **E/R del data lake**.
- **E:** **4 vistas** del frontend consumiendo **los 5 microservicios** (≥2 métodos REST c/u), incluida
  la vista de analítica (MS5); página Swagger-UI agregada; deploy final en Amplify.
- **Lead:** **diagrama de arquitectura `draw.io`** con todos los servicios AWS; **pruebas E2E** por la
  URL pública de API Gateway; verificación de que ALB y BDs son privados; consolidación de
  **informe + PPT** con evidencias de cada equipo.
- **Vie 19:** *code freeze* + ensayo de demo.

### Fase 3 — Entrega y exposición · Sáb 20 / Semana 7
- **Dom 20-Set 23:59 — Hito 2:** subir a Canvas el **informe (Word/PDF)** + **PPT** + `INDEX.md` con
  enlaces a los repos públicos.
- **Semana 7:** **exposición presencial + demo en vivo** (obligatoria).

### Camino crítico (no paralelizable)
modelos de BD → generador de data ficticia (`seeds/`) → contenedores de ingesta → S3 → crawlers Glue →
consultas/vistas Athena → **MS5** → **vista de analítica del Frontend**.
Todo lo demás (CRUD de MS1/MS2/MS3, MS4, infra de red) va en paralelo desde el Día 1 gracias a los
contratos OpenAPI.

---

## 5. Checklist Hito 1 (avance ≥50% por parte — criterio del enunciado)

- [ ] **Backend:** microservicios implementados parcialmente, **≥1 BD conectada** y consultas básicas
  funcionando. Objetivo real: MS1 + MS2 + MS3 con CRUD mínimo + MS4 devolviendo un manifiesto simple.
- [ ] **Frontend:** página inicial en Amplify (o contingencia S3+CloudFront) que consume **≥1
  microservicio con un par de métodos REST**.
- [ ] **Data Science:** **VM de ingesta configurada** + **bucket S3 creado** + **≥1 contenedor de
  ingesta funcionando con datos cargados en S3**.
- [ ] Material para la exposición virtual con el ACL (slides de avance + demo corta).

---

## 6. Checklist Hito 2 / entregables finales

- [ ] 5 microservicios en Docker desplegados en **2 VMs** con `docker compose` + **balanceador privado**
- [ ] APIs públicas por **HTTPS con API Gateway**; **3 BDs en 3ra VM privada** (sin IP pública)
- [ ] **E/R** por cada BD SQL (MS1, MS2) + **estructuras JSON** documentadas de la NoSQL (MS3); cada BD
  SQL con **≥2 tablas relacionadas**
- [ ] **≥1 microservicio consume a otro** (MS1→MS2, MS3→MS2, MS4→MS1/MS2/MS3) — con evidencia
- [ ] **1 microservicio sin BD** (MS4) · **1 microservicio analítico con Athena** (MS5)
- [ ] **Carga masiva ≥20 000 registros**, una vez, en ≥1 tabla por BD — con evidencia (`COUNT(*)`)
- [ ] **Swagger-UI** de las 5 APIs navegable
- [ ] **Frontend en Amplify** consumiendo los 5 microservicios, **≥2 métodos REST c/u**, incluida analítica
- [ ] **Data Science:** bucket S3 + **3 contenedores de ingesta (pull 100%)** + **catálogo Glue por
  archivo** + **E/R del catálogo** + **≥4 consultas Athena con join** + **≥2 vistas**
- [ ] **Diagrama de arquitectura de solución** (`draw.io`) con todos los servicios AWS
- [ ] **Informe Word/PDF** con evidencia de todo lo anterior + **PPT** resumen
- [ ] `INDEX.md` con enlaces a **todos los repos públicos**
- [ ] Exposición presencial preparada (Semana 7)

---

## 7. Próximos pasos inmediatos (antes del Sáb 6)

1. **Lead:** crear la organización GitHub + los 9 repos (README + LICENSE + plantilla) e invitar a los 5 devs.
2. **Lead:** validar en Learner Lab — EC2+VPC → S3 → Glue+Athena → API Gateway+VPC Link → Amplify — y
   escribir [`docs/aws-learner-lab-hallazgos.md`](aws-learner-lab-hallazgos.md).
3. **Todos:** subir `openapi.yaml` borrador del propio servicio + acordar
   [`docs/contratos/enums.md`](contratos/enums.md) y rangos de ID.
4. **A/B/C:** `docker compose up` local del servicio + su BD, con migraciones y 1 endpoint real.
5. **D:** escribir las 5 consultas de negocio en SQL sobre el modelo relacional (validar en Postgres local).
6. **E:** scaffold del frontend desplegado con una llamada real a `GET /api/vuelos`.
