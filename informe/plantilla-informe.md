# Informe — Proyecto Parcial CS2032 (Cloud Computing, 2026-2)

**Plantilla (EX-01).** Cada responsable redacta su sección **en F2 a medida que termina** (EX-06); Benja
consolida y exporta a **PDF** en F3 (EX-08). No dejar secciones en "TODO"; cada afirmación va con su
evidencia referenciada de [`docs/evidencias/`](../docs/evidencias/).

- **Dominio:** Aeropuerto Internacional Jorge Chávez · **Entorno:** AWS Academy Learner Lab
- **Equipo:** Benja (líder/arquitecto) · Guillermo (MS1) · Mariano (MS2) · Edinson (MS3) · Fabricio (MS4/MS5 + Data Science) · Alexander (Frontend)
- **Repos:** ver [`INDEX.md`](../INDEX.md)

Convención de evidencia: `[E-n]` en el texto → archivo en `docs/evidencias/<área>/` listado al final de la sección.

---

## 1. Introducción y objetivos · _Benja_

- Contexto del negocio y por qué se descompone el monolito de BD I en 5 microservicios.
- Objetivos del proyecto y alcance (qué entra / qué no).
- Resumen de la rúbrica y cómo el informe la cubre (tabla de trazabilidad requisito → sección).

_Evidencia:_ —

---

## 2. Arquitectura de solución · _Benja_

- Diagrama de arquitectura (PNG exportado de `diagramas/arquitectura-solucion.drawio`). **[E-2.1]**
- Lista de servicios AWS usados y su rol (tabla de [`docs/arquitectura.md`](../docs/arquitectura.md) §3).
- Las 3 capas y sus flujos (operación / persistencia / ingesta-analítica).
- Decisiones tomadas a partir de los hallazgos del Learner Lab (ALB vs NLB, Amplify vs CloudFront, VPC Link). **[E-2.2]**

_Evidencia:_ `[E-2.1]` diagrama · `[E-2.2]` `docs/aws-learner-lab-hallazgos.md`

---

## 3. Backend — 5 microservicios · _Guillermo (MS1) · Mariano (MS2) · Edinson (MS3) · Fabricio (MS4/MS5)_

Por cada microservicio (una sub-sección igual para los 5):

### 3.x MSx — <nombre> · <stack>

- Endpoints implementados (tabla) y captura de **Swagger-UI** por la URL pública. **[E-3.x.1]**
- Modelo de datos / colecciones y decisiones de diseño.
- **Consumo entre servicios:** request de ejemplo + **log** que muestra la llamada saliente (MS1→MS2, MS3→MS2, MS4→MS1/2/3). **[E-3.x.2]**
- **Volumen:** salida de `COUNT(*)` ≥ 20 000 en la tabla objetivo (`ticket` / `vuelo` / `incidencias`). **[E-3.x.3]**
- Pruebas mínimas (happy path + validación de error con el [contrato de errores](../docs/contratos/errores.md)).

### 3b. Modelos de datos

- **E/R MySQL (MS1)** y **E/R PostgreSQL (MS2)** — de [`docs/er/`](../docs/er/). **[E-3b.1]**
- **JSON Schema MongoDB (MS3)** — [`docs/er/ms3-mongo-schemas.md`](../docs/er/ms3-mongo-schemas.md). **[E-3b.2]**
- Cada BD SQL con ≥2 tablas relacionadas; se señala la relación en el diagrama.

_Evidencia:_ Swagger ×5, logs de consumo, `COUNT(*)` ×3, 2 diagramas E/R, 1 JSON Schema.

---

## 4. Transformaciones del modelo monolítico · _Fabricio_

- Del modelo relacional único (BD I) a un modelo por microservicio.
- Jerarquía `Persona` partida (IsA) entre MS1 y MS2; justificación (riesgo R8).
- FK "suaves" entre servicios (se validan por API, no por constraint) y cómo se mantiene la integridad
  (rangos de ID fijos + `seeds/` con `SEED`, ver [`contratos/enums.md`](../docs/contratos/enums.md)).
- E/R por servicio comparado con el E/R monolítico de origen.

_Evidencia:_ E/R por servicio, tabla de correspondencia entidad monolito → servicio.

---

## 5. Frontend · _Alexander_

- Las **4 vistas** con captura de cada una funcionando contra la API real. **[E-5.1]**
- **Matriz de cobertura REST**: por cada uno de los 5 MS, ≥2 métodos REST distintos, con captura del
  panel **Network** de DevTools. **[E-5.2]**
- URL pública de **Amplify** (o CloudFront si contingencia). **[E-5.3]**
- Manejo de estados de carga/error.

_Evidencia:_ 4 capturas de vistas + HAR/Network + URL.

---

## 6. Data Science · _Fabricio_

- **S3:** `aws s3 ls s3://<bucket>/raw/ --recursive` con archivos de `ms1/`, `ms2/`, `ms3/`. **[E-6.1]**
- **Ingesta:** `docker compose ps` en la VM-INGESTA con los 3 contenedores finalizados OK; logs de pull 100%. **[E-6.2]**
- **Glue:** consola con la base `aeropuerto_lake` y una tabla por archivo. **[E-6.3]**
- **E/R del catálogo:** `diagramas/er-catalogo-datalake.drawio`. **[E-6.4]**
- **Athena:** las 5 consultas con resultado (cada una con `JOIN` entre ≥2 tablas de ms1/ms2/ms3) + DDL de las 2 vistas. **[E-6.5]**

_Evidencia:_ `aws s3 ls`, `compose ps`, Glue console, E/R catálogo, 5 queries + 2 vistas.

---

## 7. Despliegue y seguridad · _Benja_

- `docker compose ps` en **VM-PROD-1** y **VM-PROD-2** (nginx + 5 MS `Up`). **[E-7.1]**
- **Verificación de "privado"** (ver [`docs/verificacion.md`](../docs/verificacion.md)):
  - `nc -zv <alb-dns> 80` desde fuera → **falla**; desde una VM-PROD → OK. **[E-7.2]**
  - `mysql -h <db-vm>` desde fuera → **falla**. **[E-7.3]**
  - Solo la URL de **API Gateway** responde por HTTPS. **[E-7.4]**
- **`RUNBOOK.md`** de reinicio tras corte (< 15 min) — ver [`aeropuerto-infra-deploy/RUNBOOK.md`](https://github.com/Cloud-MLA/aeropuerto-infra-deploy/blob/main/RUNBOOK.md). **[E-7.5]**
- Security Groups aplicados (tabla) y backups de BD a S3.

_Evidencia:_ 2× `compose ps`, 3 capturas de "falla desde fuera", URL HTTPS respondiendo, RUNBOOK.

---

## 8. Enlaces a repositorios · _Benja_

- Tabla de los **9 repos públicos** con URL y descripción — de [`INDEX.md`](../INDEX.md).
- Verificado que abren **sin login** (EX-09).

_Evidencia:_ `INDEX.md` + captura de un repo abierto en incógnito.

---

## 9. Conclusiones y limitaciones · _Benja_

- Qué se cumplió de la rúbrica (checklist).
- **Contingencias aplicadas** y por qué (Amplify→CloudFront, ALB→NLB, VPC Link, `t3.small`→`t3.medium`, etc.).
- Limitaciones del Learner Lab (temporizador, `LabRole`, presupuesto) y su impacto.
- Trabajo futuro.

_Evidencia:_ —

---

## Anexo — Trazabilidad rúbrica → evidencia

| Requisito de la rúbrica | Sección | Evidencia |
|---|---|---|
| 5 microservicios, 3 lenguajes, 3 BD | 3 | `[E-3.*]` |
| ≥1 microservicio consume a otro | 3 | `[E-3.x.2]` |
| 1 sin BD + 1 analítico Athena | 3 (MS4, MS5) | — |
| ≥20 000 registros por BD | 3 | `[E-3.x.3]` |
| APIs solo por API Gateway HTTPS; ALB/BD privados | 7 | `[E-7.2..7.4]` |
| Swagger-UI de las 5 + agregada | 3, 5 | `[E-3.x.1]` |
| E/R MySQL + PostgreSQL + JSON Schema Mongo | 3b | `[E-3b.*]` |
| Frontend 4 vistas, 5 MS, ≥2 métodos c/u | 5 | `[E-5.*]` |
| Data lake: S3 + 3 ingestas + Glue + ≥4 Athena + ≥2 vistas + E/R catálogo | 6 | `[E-6.*]` |
| Diagrama de arquitectura | 2 | `[E-2.1]` |
| Despliegue `compose` en 2 VM + RUNBOOK | 7 | `[E-7.1]`, `[E-7.5]` |
| Repos públicos + INDEX | 8 | — |
