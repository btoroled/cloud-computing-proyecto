# Distribución del trabajo

Vista consolidada de quién hace qué, a través de todas las secciones. Detalle de tareas en
[`backend.md`](backend.md) · [`frontend.md`](frontend.md) · [`data-science.md`](data-science.md) ·
[`diagrama-arquitectura.md`](diagrama-arquitectura.md) · [`exposiciones-e-informe.md`](exposiciones-e-informe.md).
Checklist personal por integrante en [`personas/`](personas/README.md).

Equipo: **6 personas** — 1 Líder / Arquitecto (Benja) + Guillermo, Mariano, Edinson, Fabricio, Alexander (mismo nivel).
Convención: **d** = días-persona (≈ 3–4 h). Ventana útil F0–F2 ≈ 12 días hábiles.

---

## 1. Repos que posee cada persona

| Persona | Repos que lidera |
|---|---|
| **Benja** | `aeropuerto-infra-deploy`, `cloud-computing-proyecto` (docs) |
| **Guillermo** | `ms1-pasajeros-api` + contenedor `ingesta-ms1` (en `aeropuerto-data-science`) |
| **Mariano** | `ms2-vuelos-api` + contenedor `ingesta-ms2` |
| **Edinson** | `ms3-infraestructura-api` + contenedor `ingesta-ms3` |
| **Fabricio** | `ms4-manifiesto-api`, `ms5-analitica-api`, `aeropuerto-data-science` (Glue/Athena/seeds) |
| **Alexander** | `aeropuerto-frontend` |

---

## 2. Matriz RACI por entregable de la rúbrica

R = responsable ejecuta · A = aprueba/valida · C = consultado · I = informado.

| Entregable | R | A | C | I |
|---|---|---|---|---|
| MS1 Pasajeros/Tickets | Guillermo | Benja | Fabricio | equipo |
| MS2 Vuelos/Operaciones | Mariano | Benja | Guillermo, Edinson | equipo |
| MS3 Infraestructura/Incidencias | Edinson | Benja | Fabricio | equipo |
| MS4 Manifiesto (sin BD) | Fabricio | Benja | Guillermo, Mariano, Edinson | equipo |
| MS5 Analítico (Athena) | Fabricio | Benja | Alexander | equipo |
| `seeds/` datos ficticios (20k) | Fabricio | Benja | Guillermo, Mariano, Edinson | equipo |
| Ingesta — 3 contenedores | Guillermo / Mariano / Edinson (uno c/u) | Fabricio | Benja | equipo |
| Bucket S3 + VM-INGESTA | Benja | — | Fabricio | equipo |
| Catálogo Glue | Fabricio | Benja | Benja (permisos) | equipo |
| Consultas + vistas Athena | Fabricio | Benja | — | equipo |
| E/R del catálogo de datos | Alexander | Fabricio | Guillermo/Mariano/Edinson | equipo |
| E/R MySQL / PostgreSQL | Guillermo / Mariano | Benja | Fabricio | equipo |
| JSON Schema MongoDB | Edinson | Benja | Fabricio | equipo |
| Frontend SPA (4 vistas) | Alexander | Benja | Fabricio | equipo |
| Deploy Amplify / contingencia | Alexander | Benja | — | equipo |
| Página Swagger-UI agregada | Alexander | Benja | Benja (nginx) | equipo |
| VPC / EC2 / SG / ALB / API Gateway + VPC Link | Benja | — | Mariano | equipo |
| Deploy `docker compose` en 2 VM-PROD + VM-DB | Benja | — | Guillermo/Mariano/Edinson | equipo |
| `RUNBOOK.md` + backups a S3 | Benja | — | — | equipo |
| Diagrama de arquitectura | Benja | equipo | Fabricio, Alexander | — |
| Prueba de humo E2E + verificación de "privado" | Benja | — | — | equipo |
| Slides Hito 1 (ACL) | Benja (consolida) | equipo | todos | — |
| Informe final | Benja (consolida) | equipo | todos (cada quien su sección) | — |
| PPT resumen | Benja + todos | equipo | — | — |
| Exposición virtual ACL / presencial | todos | — | — | — |

---

## 3. Carga estimada por persona y fase

| Persona | F0 (Mié 2–Vie 4) | F1 (Sáb 6–Sáb 12) | F2 (Sáb 13–Vie 19) | F3 | Total ≈ |
|---|---|---|---|---|---|
| **Benja** | Repos + Learner Lab + VPC/SG/EC2 + boceto diagrama (~3.5d) | VM-DB + compose 2 VM + ALB + API Gateway/VPC Link + nginx + slides H1 (~5d) | Diagrama v2 + E2E + logs + consolidar informe/PPT (~5.5d) | Subida Canvas + demo (~1d) | **~15 d** |
| **Guillermo** | Scaffold MS1 + modelo/migraciones + OpenAPI (~2d) | CRUD MS1 + MS1→MS2 + checkin/equipaje + `ingesta-ms1` + seed 5k (~5d) | 20k + Swagger + tests + E/R MySQL + sección informe (~3d) | — | **~10 d** |
| **Mariano** ⚠️ | Scaffold MS2 (Java) + entidades/Flyway + OpenAPI (~2.5d) | CRUD vuelos/aerolíneas/aeronaves + `/exists` + tripulación + seed 2k (~5.5d) | `ingesta-ms2` + 20k `vuelo`/`asiento` + tuning JVM + Swagger + E/R PG + informe (~4d) | — | **~12 d** |
| **Edinson** | Scaffold MS3 + JSON Schema + OpenAPI (~2d) | CRUD recursos/incidencias/asignaciones + MS3→MS2 + seed 5k (~4.5d) | `ingesta-ms3` con aplanado + 20k `incidencias` + Swagger + informe (~4d) | — | **~10.5 d** |
| **Fabricio** ⚠️ | Estructura data-science + 5 queries en SQL local + `seeds/` v1 (~3d) | MS4 `/manifiesto` + `seeds/` final + ayuda ingesta-ms1 (~4d) | MS5 (5 endpoints) + Glue + Athena Q1–Q5 + 2 vistas + transformaciones informe (~7d) | — | **~14 d** |
| **Alexander** ⚠️ | Scaffold React + cliente HTTP + pipeline Amplify + mocks (~2.5d) | Vista "Consulta de vuelo + manifiesto" + deploy H1 (~2d) | 3 vistas + Dashboard + Swagger agregado + E/R catálogo + pulido (~8d) | — | **~12.5 d** |

⚠️ = personas en el límite de capacidad. Ver rebalanceo en §5.

---

## 4. Qué entrega cada persona en cada hito

| Persona | Hito 1 (Sáb 12) | Hito 2 (Dom 20) |
|---|---|---|
| **Benja** | 2 VM-PROD + ALB privado + API Gateway respondiendo; slides y demo H1 consolidadas; boceto de diagrama | Despliegue completo verificado (privado); diagrama final; informe + PPT consolidados; subida a Canvas |
| **Guillermo** | MS1 con CRUD mínimo + MS1→MS2 + BD MySQL conectada; `ingesta-ms1` cargando a S3 | MS1 completo + 20k en `ticket`/`equipaje` + Swagger + E/R MySQL |
| **Mariano** | MS2 con CRUD mínimo + `/vuelos/{id}/exists` + BD PostgreSQL conectada | MS2 completo + 20k en `vuelo`/`asiento` + Swagger + E/R PostgreSQL + `ingesta-ms2` |
| **Edinson** | MS3 con CRUD mínimo + MS3→MS2 + BD MongoDB conectada | MS3 completo + 20k en `incidencias` + Swagger + JSON Schema + `ingesta-ms3` |
| **Fabricio** | MS4 devolviendo manifiesto simple; `seeds/` operativo; 5 queries validadas en local | MS5 con 5 endpoints Athena; Glue + 2 vistas; E/R catálogo; sección de transformaciones |
| **Alexander** | SPA en Amplify consumiendo MS2 con ≥2 métodos REST (vista de vuelo/manifiesto) | 4 vistas completas consumiendo los 5 MS; Swagger agregado; capturas de cobertura REST |

---

## 5. Cuellos de botella y rebalanceo

**Personas más cargadas:** Fabricio (~14d), Alexander (~12.5d), Mariano (~12d).

Ajustes sugeridos (aplicar en el standup según avance real):

1. **`seeds/`**: en vez de que Fabricio lo haga todo, **cada dev genera el seed de su propia BD** y Fabricio
   solo define el contrato de IDs y ejecuta la orquestación. Descarga ~1d de Fabricio.
2. **E/R del catálogo de datos (DS-13)**: asignado a **Alexander** (es diagramación, no código) → libera a Fabricio.
3. **MS5**: si Guillermo o Edinson terminan su microservicio antes de F2, uno de ellos toma MS5 (Python + boto3,
   relativamente simple una vez que Athena está lista). Descarga ~3d de Fabricio.
4. **Dashboard (FE-10)**: Fabricio aporta las consultas/formato de datos y Alexander solo maqueta → trabajo
   compartido ya previsto.
5. **Tablas de volumen de MS2** (`asiento`, `opera_tripulacion`): si MS3 va adelantado, **Edinson ayuda a
   Mariano** con el generador de esas tablas.
6. **Pulido del Frontend (FE-12)**: es lo primero que se recorta si Alexander no llega; la rúbrica valora
   que consuma los 5 MS, no el diseño.

---

## 6. Dependencias entre personas (quién espera a quién)

```
Benja (infra F0/F1) ──────────► todos (deploy, API Gateway)
Mariano (MS2 /vuelos/{id}/exists) ──► Guillermo (POST /tickets), Edinson (POST /incidencias), Fabricio (MS4)
Guillermo/Mariano/Edinson (ingesta) ──► Fabricio (Glue) ──► Fabricio (Athena) ──► Fabricio (MS5) ──► Alexander (Dashboard)
Fabricio (seeds/ + contrato de IDs) ──► Guillermo/Mariano/Edinson (carga 20k coherente)
Benja (URL API Gateway) ──► Alexander (frontend)   [mitigable: Alexander consume MS2 directo en Hito 1]
```

**Regla de desbloqueo:** todo el mundo trabaja **contract-first** desde F0. Si una dependencia no está
lista, se usa un *mock* del `openapi.yaml` y se integra después. Benja reasigna en el standup diario.

---

## 7. Contingencia de personas

| Situación | Respuesta |
|---|---|
| Un dev no entrega su microservicio a tiempo | Los consumidores siguen con *mock*; Benja reasigna la tarea o reduce su alcance al mínimo de rúbrica |
| Fabricio saturado (MS4+MS5+DS) | Aplicar rebalanceos §5 puntos 1–3 de inmediato |
| Alexander saturado (Frontend) | Recortar FE-12 (pulido); Fabricio toma FE-10; 3 vistas + dashboard es el mínimo |
| Benja bloqueado en infra (Learner Lab) | Mariano apoya en la parte de red/EC2; se prioriza el camino "1 BD conectada + 1 MS" para Hito 1 |
| Ausencia imprevista en la exposición presencial | Los 6 deben confirmar asistencia el Vie 19; tener video de respaldo de la demo |
