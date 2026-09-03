# Distribución del trabajo

Vista consolidada de quién hace qué, a través de todas las secciones. Detalle de tareas en
[`backend.md`](backend.md) · [`frontend.md`](frontend.md) · [`data-science.md`](data-science.md) ·
[`diagrama-arquitectura.md`](diagrama-arquitectura.md) · [`exposiciones-e-informe.md`](exposiciones-e-informe.md).

Equipo: **6 personas** — 1 Lead/Arquitecto (Benjamín) + Dev A, Dev B, Dev C, Dev D, Dev E (mismo nivel).
Convención: **d** = días-persona (≈ 3–4 h). Ventana útil F0–F2 ≈ 12 días hábiles.

---

## 1. Repos que posee cada persona

| Persona | Repos que lidera |
|---|---|
| **Lead** | `aeropuerto-infra-deploy`, `cloud-computing-proyecto` (docs) |
| **Dev A** | `ms1-pasajeros-api` + contenedor `ingesta-ms1` (en `aeropuerto-data-science`) |
| **Dev B** | `ms2-vuelos-api` + contenedor `ingesta-ms2` |
| **Dev C** | `ms3-infraestructura-api` + contenedor `ingesta-ms3` |
| **Dev D** | `ms4-manifiesto-api`, `ms5-analitica-api`, `aeropuerto-data-science` (Glue/Athena/seeds) |
| **Dev E** | `aeropuerto-frontend` |

---

## 2. Matriz RACI por entregable de la rúbrica

R = responsable ejecuta · A = aprueba/valida · C = consultado · I = informado.

| Entregable | R | A | C | I |
|---|---|---|---|---|
| MS1 Pasajeros/Tickets | Dev A | Lead | Dev D | equipo |
| MS2 Vuelos/Operaciones | Dev B | Lead | Dev A, Dev C | equipo |
| MS3 Infraestructura/Incidencias | Dev C | Lead | Dev D | equipo |
| MS4 Manifiesto (sin BD) | Dev D | Lead | Dev A, Dev B, Dev C | equipo |
| MS5 Analítico (Athena) | Dev D | Lead | Dev E | equipo |
| `seeds/` datos ficticios (20k) | Dev D | Lead | Dev A, Dev B, Dev C | equipo |
| Ingesta — 3 contenedores | Dev A / Dev B / Dev C (uno c/u) | Dev D | Lead | equipo |
| Bucket S3 + VM-INGESTA | Lead | — | Dev D | equipo |
| Catálogo Glue | Dev D | Lead | Lead (permisos) | equipo |
| Consultas + vistas Athena | Dev D | Lead | — | equipo |
| E/R del catálogo de datos | Dev E | Dev D | Dev A/B/C | equipo |
| E/R MySQL / PostgreSQL | Dev A / Dev B | Lead | Dev D | equipo |
| JSON Schema MongoDB | Dev C | Lead | Dev D | equipo |
| Frontend SPA (4 vistas) | Dev E | Lead | Dev D | equipo |
| Deploy Amplify / contingencia | Dev E | Lead | — | equipo |
| Página Swagger-UI agregada | Dev E | Lead | Lead (nginx) | equipo |
| VPC / EC2 / SG / ALB / API Gateway + VPC Link | Lead | — | Dev B | equipo |
| Deploy `docker compose` en 2 VM-PROD + VM-DB | Lead | — | Dev A/B/C | equipo |
| `RUNBOOK.md` + backups a S3 | Lead | — | — | equipo |
| Diagrama de arquitectura | Lead | equipo | Dev D, Dev E | — |
| Prueba de humo E2E + verificación de "privado" | Lead | — | — | equipo |
| Slides Hito 1 (ACL) | Lead (consolida) | equipo | todos | — |
| Informe final | Lead (consolida) | equipo | todos (cada quien su sección) | — |
| PPT resumen | Lead + todos | equipo | — | — |
| Exposición virtual ACL / presencial | todos | — | — | — |

---

## 3. Carga estimada por persona y fase

| Persona | F0 (Mié 2–Vie 4) | F1 (Sáb 6–Sáb 12) | F2 (Sáb 13–Vie 19) | F3 | Total ≈ |
|---|---|---|---|---|---|
| **Lead** | Repos + Learner Lab + VPC/SG/EC2 + boceto diagrama (~3.5d) | VM-DB + compose 2 VM + ALB + API Gateway/VPC Link + nginx + slides H1 (~5d) | Diagrama v2 + E2E + logs + consolidar informe/PPT (~5.5d) | Subida Canvas + demo (~1d) | **~15 d** |
| **Dev A** | Scaffold MS1 + modelo/migraciones + OpenAPI (~2d) | CRUD MS1 + MS1→MS2 + checkin/equipaje + `ingesta-ms1` + seed 5k (~5d) | 20k + Swagger + tests + E/R MySQL + sección informe (~3d) | — | **~10 d** |
| **Dev B** ⚠️ | Scaffold MS2 (Java) + entidades/Flyway + OpenAPI (~2.5d) | CRUD vuelos/aerolíneas/aeronaves + `/exists` + tripulación + seed 2k (~5.5d) | `ingesta-ms2` + 20k `vuelo`/`asiento` + tuning JVM + Swagger + E/R PG + informe (~4d) | — | **~12 d** |
| **Dev C** | Scaffold MS3 + JSON Schema + OpenAPI (~2d) | CRUD recursos/incidencias/asignaciones + MS3→MS2 + seed 5k (~4.5d) | `ingesta-ms3` con aplanado + 20k `incidencias` + Swagger + informe (~4d) | — | **~10.5 d** |
| **Dev D** ⚠️ | Estructura data-science + 5 queries en SQL local + `seeds/` v1 (~3d) | MS4 `/manifiesto` + `seeds/` final + ayuda ingesta-ms1 (~4d) | MS5 (5 endpoints) + Glue + Athena Q1–Q5 + 2 vistas + transformaciones informe (~7d) | — | **~14 d** |
| **Dev E** ⚠️ | Scaffold React + cliente HTTP + pipeline Amplify + mocks (~2.5d) | Vista "Consulta de vuelo + manifiesto" + deploy H1 (~2d) | 3 vistas + Dashboard + Swagger agregado + E/R catálogo + pulido (~8d) | — | **~12.5 d** |

⚠️ = personas en el límite de capacidad. Ver rebalanceo en §5.

---

## 4. Qué entrega cada persona en cada hito

| Persona | Hito 1 (Sáb 12) | Hito 2 (Dom 20) |
|---|---|---|
| **Lead** | 2 VM-PROD + ALB privado + API Gateway respondiendo; slides y demo H1 consolidadas; boceto de diagrama | Despliegue completo verificado (privado); diagrama final; informe + PPT consolidados; subida a Canvas |
| **Dev A** | MS1 con CRUD mínimo + MS1→MS2 + BD MySQL conectada; `ingesta-ms1` cargando a S3 | MS1 completo + 20k en `ticket`/`equipaje` + Swagger + E/R MySQL |
| **Dev B** | MS2 con CRUD mínimo + `/vuelos/{id}/exists` + BD PostgreSQL conectada | MS2 completo + 20k en `vuelo`/`asiento` + Swagger + E/R PostgreSQL + `ingesta-ms2` |
| **Dev C** | MS3 con CRUD mínimo + MS3→MS2 + BD MongoDB conectada | MS3 completo + 20k en `incidencias` + Swagger + JSON Schema + `ingesta-ms3` |
| **Dev D** | MS4 devolviendo manifiesto simple; `seeds/` operativo; 5 queries validadas en local | MS5 con 5 endpoints Athena; Glue + 2 vistas; E/R catálogo; sección de transformaciones |
| **Dev E** | SPA en Amplify consumiendo MS2 con ≥2 métodos REST (vista de vuelo/manifiesto) | 4 vistas completas consumiendo los 5 MS; Swagger agregado; capturas de cobertura REST |

---

## 5. Cuellos de botella y rebalanceo

**Personas más cargadas:** Dev D (~14d), Dev E (~12.5d), Dev B (~12d).

Ajustes sugeridos (aplicar en el standup según avance real):

1. **`seeds/`**: en vez de que Dev D lo haga todo, **cada dev genera el seed de su propia BD** y Dev D
   solo define el contrato de IDs y ejecuta la orquestación. Descarga ~1d de Dev D.
2. **E/R del catálogo de datos (DS-13)**: asignado a **Dev E** (es diagramación, no código) → libera a Dev D.
3. **MS5**: si Dev A o Dev C terminan su microservicio antes de F2, uno de ellos toma MS5 (Python + boto3,
   relativamente simple una vez que Athena está lista). Descarga ~3d de Dev D.
4. **Dashboard (FE-10)**: Dev D aporta las consultas/formato de datos y Dev E solo maqueta → trabajo
   compartido ya previsto.
5. **Tablas de volumen de MS2** (`asiento`, `opera_tripulacion`): si MS3 va adelantado, **Dev C ayuda a
   Dev B** con el generador de esas tablas.
6. **Pulido del Frontend (FE-12)**: es lo primero que se recorta si Dev E no llega; la rúbrica valora
   que consuma los 5 MS, no el diseño.

---

## 6. Dependencias entre personas (quién espera a quién)

```
Lead (infra F0/F1) ──────────► todos (deploy, API Gateway)
Dev B (MS2 /vuelos/{id}/exists) ──► Dev A (POST /tickets), Dev C (POST /incidencias), Dev D (MS4)
Dev A/B/C (ingesta) ──► Dev D (Glue) ──► Dev D (Athena) ──► Dev D (MS5) ──► Dev E (Dashboard)
Dev D (seeds/ + contrato de IDs) ──► Dev A/B/C (carga 20k coherente)
Lead (URL API Gateway) ──► Dev E (frontend)   [mitigable: Dev E consume MS2 directo en Hito 1]
```

**Regla de desbloqueo:** todo el mundo trabaja **contract-first** desde F0. Si una dependencia no está
lista, se usa un *mock* del `openapi.yaml` y se integra después. El Lead reasigna en el standup diario.

---

## 7. Contingencia de personas

| Situación | Respuesta |
|---|---|
| Un dev no entrega su microservicio a tiempo | Los consumidores siguen con *mock*; el Lead reasigna la tarea o reduce su alcance al mínimo de rúbrica |
| Dev D saturado (MS4+MS5+DS) | Aplicar rebalanceos §5 puntos 1–3 de inmediato |
| Dev E saturado (Frontend) | Recortar FE-12 (pulido); Dev D toma FE-10; 3 vistas + dashboard es el mínimo |
| Lead bloqueado en infra (Learner Lab) | Dev B apoya en la parte de red/EC2; se prioriza el camino "1 BD conectada + 1 MS" para Hito 1 |
| Ausencia imprevista en la exposición presencial | Los 6 deben confirmar asistencia el Vie 19; tener video de respaldo de la demo |
