# Requerimientos del Proyecto

Proyecto Parcial CS2032 — Cloud Computing (Ciclo 2026-2). Dominio: Aeropuerto Internacional Jorge Chávez.

Documentos relacionados: [plan de trabajo](plan-de-trabajo.md) · [arquitectura](arquitectura.md) ·
[enums](contratos/enums.md) · [riesgos](riesgos.md) · [verificación](verificacion.md).

---

## 1. Alcance y mapeo a la rúbrica

| Componente | Puntos | Entregable concreto | Hito |
|---|---|---|---|
| Backend — 5 microservicios en Docker | 7 | MS1..MS5 desplegados en 2 VMs con `docker compose` + balanceador privado + API Gateway HTTPS + BDs en 3ra VM privada + Swagger-UI | H2 |
| Frontend — Web | 3 | SPA React en AWS Amplify que consume los 5 microservicios (≥2 métodos REST c/u) | H2 |
| Data Science — Analytics | 5 | Bucket S3 + 3 contenedores de ingesta (pull 100%) + catálogo Glue + E/R del catálogo + ≥4 consultas Athena con join + ≥2 vistas | H2 |
| Diagrama de arquitectura de solución | 1 | `draw.io` con todos los servicios AWS de Backend + Frontend + Data Science | H2 |
| Exposición presencial + demo | 1 | Demo en vivo Semana 7 (**obligatoria**: si no se presenta, nota desaprobatoria tope 10) | Sem 7 |
| Exposición virtual con ACL | 3 | Revisión de avance ≥50% por parte con el ACL | **H1** |

**Reparto de nota:** Hito 1 = 3 pts · Hito 2 = 17 pts. **Total = 20 pts.**

### Requisitos textuales del enunciado (checklist maestro)

**Backend**
- 5 microservicios en Docker.
- 3 microservicios con BD propia, en **3 lenguajes distintos** y **3 BD distintas** (2 SQL + 1 NoSQL).
- Cada BD SQL con **mínimo 2 tablas relacionadas**.
- Diagrama **E/R** por cada BD SQL + **estructuras JSON** de la BD NoSQL.
- **Al menos 1 microservicio consume a otro.**
- 1 microservicio **sin BD** que solo consume otros.
- 1 microservicio **analítico** que ejecuta queries con **Athena**.
- **Carga masiva, por única vez, de datos ficticios: mínimo 20 000 registros** en al menos 1 tabla de
  cada BD.
- Despliegue con `docker compose` en **2 VMs de producción** con **balanceador de carga privado**
  (no público).
- APIs expuestas públicamente con **HTTPS** vía **AWS API Gateway**.
- Las BDs en una **3ra VM privada** (no pública).
- Documentar las 5 APIs en **Swagger-UI**.
- Enlaces a repositorios públicos de GitHub.

**Frontend**
- Página web que consume **los 5 microservicios**, incluido el analítico.
- **Al menos 2 métodos REST por microservicio.**
- Desplegada en **AWS Amplify**.
- Enlaces a repositorios públicos.

**Data Science**
- Crear un **bucket S3** para los archivos de ingesta.
- **3 contenedores Docker en Python** para ingesta con **estrategia pull del 100%** de los registros;
  cada contenedor ingesta 1 microservicio y genera CSV/JSON que carga en S3.
- **Catálogo de datos en AWS Glue** por cada archivo cargado.
- **Diagrama E/R** que relacione todas las tablas del catálogo.
- Evidencia de **mínimo 4 consultas SQL** que unan varias tablas con **AWS Athena** + crear **mínimo
  2 vistas**.
- Enlaces a repositorios públicos.

**Diagrama de arquitectura de solución**
- En `draw.io`, incluyendo **todos** los servicios de AWS usados en Backend + Frontend + Data Science.

---

## 2. Descomposición en microservicios

| # | Nombre | Lenguaje / framework | Tipo de BD | Motor | Consume a | Repo |
|---|---|---|---|---|---|---|
| MS1 | Pasajeros / Tickets | Python + FastAPI | Relacional SQL | **MySQL 8** | MS2 | `ms1-pasajeros-api` |
| MS2 | Vuelos / Operaciones | Java + Spring Boot | Relacional SQL | **PostgreSQL 16** | — | `ms2-vuelos-api` |
| MS3 | Infraestructura / Incidencias | Node.js + Express | Documental NoSQL | **MongoDB 7** | MS2 | `ms3-infraestructura-api` |
| MS4 | Manifiesto de Vuelo | Python + FastAPI | **Sin BD** | MS1 + MS2 + MS3 | — | `ms4-manifiesto-api` |
| MS5 | Analítico | Python + FastAPI + boto3 | **Sin BD** (data lake S3/Glue/Athena) | — (Athena) | `ms5-analitica-api` |

Cumple: 3 lenguajes distintos · 3 BD distintas (2 SQL + 1 NoSQL) · cada BD SQL con ≥2 tablas
relacionadas · ≥1 microservicio consume a otro · 1 microservicio sin BD · 1 microservicio analítico
con Athena.

### 2.1 Reparto de las 21 relaciones del modelo del aeropuerto

**MS1 — MySQL** (`pasajeros_db`)
Tablas: `persona`, `pasajero`, `categoria_migratoria`, `ticket`, `checkin`, `equipaje`.
- Relaciones: `pasajero` 1–N `ticket`; `ticket` 1–1 `checkin`; `pasajero` 1–N `equipaje`;
  `categoria_migratoria` 1–N `pasajero`; `persona` 1–1 `pasajero` (jerarquía IsA, ver §3.1).
- Tabla de volumen ≥20 000: `ticket` y `equipaje`.
- `ticket.vuelo_id` y `equipaje.vuelo_id` = **referencia suave** a MS2 (sin FK física).

**MS2 — PostgreSQL** (`vuelos_db`)
Tablas: `vuelo`, `aerolinea`, `aeronave`, `asiento`, `empleado`, `tripulacion`, `operativo_tierra`,
`opera_tripulacion`.
- `empleado` reemplaza a `persona` para el personal (identidad propia del servicio, ver §3.1).
- Relaciones: `aerolinea` 1–N `vuelo`; `aeronave` 1–N `vuelo`; `aeronave` 1–N `asiento` (entidad
  débil); `empleado` 1–1 `tripulacion` / `operativo_tierra` (IsA); `tripulacion` N–M `vuelo` vía
  `opera_tripulacion`.
- Tabla de volumen ≥20 000: `vuelo` y `asiento`.
- Claves naturales conservadas: `aerolinea.ruc` (SUNAT), `aeronave.placa` (DGAC).

**MS3 — MongoDB** (`infra_db`)
Colecciones: `recursos`, `incidencias`, `asignaciones`.
- `recursos`: documento con `tipo: "manga" | "radar"` y subdocumento con los atributos del subtipo.
- `incidencias`: documento con arrays `afecta_recursos[]` y `retrasa_vuelos[]`.
- `asignaciones`: relación `Utiliza` (vuelo ↔ recurso); MS3 valida `vuelo_id` contra MS2.
- Colección de volumen ≥20 000: `incidencias` (el modelo original la proyecta a millones).
- Los `vuelo_id` referenciados son **referencia suave** a MS2.

Ejemplo de estructura JSON de `recursos` (manga):

```json
{
  "_id": "665f...",
  "id": 42,
  "nombre_tecnico_locacion": "Espigón A - Puente A07",
  "tipo": "manga",
  "manga": { "estado_acople": "Libre", "longitud": 24.5, "clase_max": "E" }
}
```

Ejemplo de estructura JSON de `incidencias`:

```json
{
  "_id": "665f...",
  "id": 10231,
  "gravedad": "Alta",
  "descripcion": "Falla intermitente del radar primario en banda S",
  "tipo_incidencia": "Falla_Radar",
  "fecha_reporte": "2026-09-01T14:22:00Z",
  "fecha_cierre": null,
  "afecta_recursos": [ { "recurso_id": 7 } ],
  "retrasa_vuelos": [ { "vuelo_id": 1841 }, { "vuelo_id": 1843 } ]
}
```

---

## 3. Transformaciones del modelo monolítico → microservicios

*(Se documentan en el informe como "Especificaciones de Transformación".)*

### 3.1 Jerarquía `Persona` partida entre servicios
En el modelo original `Persona` es superclase de `Pasajero` **y** de `Personal → Tripulacion /
Operativo-Tierra`. En una arquitectura de microservicios no se comparte tabla:

- **MS1** es dueño de `persona` / `pasajero` — identidad de los viajeros.
- **MS2** tiene su propia tabla `empleado` con `nombre`, `apellido`, `fecha_nacimiento` embebidos, y
  sus subclases `tripulacion` (`num_licencia`) / `operativo_tierra` (`area_operativa`).

El E/R **por microservicio** refleja esta duplicación deliberada de los atributos de identidad. Es una
desviación consciente del modelo relacional único y se explica en el informe.

### 3.2 FK entre servicios → referencias suaves validadas por REST
`ticket.vuelo_id`, `equipaje.vuelo_id` (MS1→MS2), `incidencia.retrasa_vuelos[]`, `asignacion.vuelo_id`
(MS3→MS2) **no** son `FOREIGN KEY`. Se validan con una llamada REST al escribir. Esto materializa el
requisito de "un microservicio consume a otro":

- **MS1 → MS2**: al `POST /tickets`, MS1 llama `GET /api/vuelos/{id}/exists`. Si el vuelo no existe o
  está `Cancelado`, MS1 responde `422`.
- **MS3 → MS2**: al `POST /incidencias` con vuelos retrasados o al `POST /asignaciones`, MS3 valida los
  `vuelo_id` contra MS2.
- **MS4 → MS1 + MS2 + MS3**: agregación pura (ver §4.4).

### 3.3 Otras notas de transformación
- `asiento` queda con FK solo a `aeronave` (el modelo original no liga `ticket`↔`asiento`). Sirve como
  tabla de volumen en MS2. *Opcional:* añadir `asiento_codigo` a `ticket` para realismo (no requerido).
- Enums: PostgreSQL usa tipos `ENUM` nativos; MySQL usa `ENUM` / `CHECK`; MongoDB valida en la capa de
  aplicación. Los **strings** deben ser idénticos en los 3 servicios para que los joins de Athena
  funcionen — ver [`contratos/enums.md`](contratos/enums.md).
- Rangos de ID deterministas para preservar integridad entre servicios: ver
  [plan de trabajo §"Datos de prueba"](plan-de-trabajo.md#datos-de-prueba-integridad-entre-servicios).

---

## 4. Requerimientos funcionales por componente

> Regla transversal: **cada microservicio debe ser invocado por el frontend con ≥2 métodos REST
> distintos.** Cada servicio expone `GET /health` y su OpenAPI en `/openapi.json` + Swagger-UI en `/docs`.

### 4.1 MS1 — Pasajeros / Tickets (Python + FastAPI + MySQL) · base `/api/pasajeros`
- `POST /pasajeros` — crear pasajero (clave alterna: `tipo_documento` + `numero_documento`)
- `GET /pasajeros/{id}` · `GET /pasajeros?tipo_documento=&numero_documento=`
- `GET /categorias-migratorias` — lista categorías con tarifa TUUA
- `POST /tickets` — emitir ticket (**llama a MS2** para validar el vuelo)
- `GET /tickets/{id}` · `GET /tickets?vuelo_id=` · `GET /pasajeros/{id}/tickets`
- `POST /tickets/{id}/checkin` — registrar check-in (1:1 con ticket)
- `POST /equipajes` · `GET /equipajes?pasajero_id=` · `GET /equipajes?vuelo_id=`

### 4.2 MS2 — Vuelos / Operaciones (Java + Spring Boot + PostgreSQL) · base `/api/vuelos`
- `GET /vuelos/{id}` · `GET /vuelos?num=&estado=&tipo=&fecha=` · `POST /vuelos`
- `PATCH /vuelos/{id}/estado` — transición de estado operativo
- `GET /vuelos/{id}/exists` — endpoint liviano de validación para MS1 y MS3
- `GET /aerolineas` · `GET /aerolineas/{ruc}` · `POST /aerolineas`
- `GET /aeronaves` · `GET /aeronaves/{placa}` · `GET /aeronaves/{placa}/asientos`
- `GET /empleados/{id}` · `GET /empleados?tipo=Tripulacion`
- `POST /vuelos/{id}/tripulacion` · `GET /vuelos/{id}/tripulacion`

### 4.3 MS3 — Infraestructura / Incidencias (Node.js + Express + MongoDB) · base `/api/infra`
- `GET /recursos` · `GET /recursos/{id}` · `GET /recursos?tipo=manga&estado=Libre`
- `POST /recursos` (manga | radar) · `PATCH /recursos/{id}/estado`
- `POST /incidencias` (**valida vuelos contra MS2**) · `GET /incidencias` ·
  `GET /incidencias?tipo=&desde=&hasta=` · `GET /incidencias/{id}` · `PATCH /incidencias/{id}/cierre`
- `POST /asignaciones` (Utiliza: vuelo ↔ recurso) · `GET /asignaciones?vuelo_id=` ·
  `GET /asignaciones?recurso_id=`

### 4.4 MS4 — Manifiesto de Vuelo (Python + FastAPI, sin BD) · base `/api/manifiesto`
- `GET /manifiesto/{vuelo_id}` — consolida: datos del vuelo + aeronave + aerolínea (MS2) · lista de
  pasajeros con ticket / estado de check-in / equipaje (MS1) · tripulación asignada (MS2) · recursos
  asignados + incidencias abiertas del vuelo (MS3)
- `GET /manifiesto/{vuelo_id}/pasajeros` · `GET /manifiesto/{vuelo_id}/resumen` (conteos: pax, equipaje
  total kg, incidencias abiertas)
- `GET /health/dependencias` — estado de MS1 / MS2 / MS3

### 4.5 MS5 — Analítico (Python + FastAPI + boto3 → Athena) · base `/api/analitica`
Cada endpoint ejecuta una consulta Athena nombrada (o lee una vista):
- `GET /analitica/recursos-mas-fallas?dias=7` — recurso con más incidencias en la última semana
- `GET /analitica/retraso-promedio?tipo=Internacional` — retraso medio de vuelos internacionales
- `GET /analitica/incidencias-combustible-por-aerolinea`
- `GET /analitica/recaudacion-tuua-por-categoria` — recaudación estimada TUUA por categoría migratoria
- `GET /analitica/vuelos-hora-punta-retrasados` — % de vuelos en hora punta (06–09 h / 18–21 h)
  retrasados

Las 5 consultas cubren el mínimo de ≥4 con join. ≥2 de ellas se materializan como vistas
(`vw_recaudacion_tuua`, `vw_retrasos_hora_punta`).

### 4.6 Frontend — SPA React (repo `aeropuerto-frontend`, deploy Amplify)
Vistas (cada una fija el mínimo de ≥2 métodos REST por microservicio):
- **Dashboard de crisis** — consume MS5 (5 indicadores / gráficos)
- **Consulta de vuelo + manifiesto** — consume MS4 (`GET /manifiesto/{id}`, `GET .../resumen`) y MS2
  (`GET /vuelos`, `GET /aeronaves/{placa}/asientos`)
- **Recursos e incidencias** — consume MS3 (`GET /recursos?estado=Libre`, `POST /incidencias`,
  `GET /incidencias`)
- **Emisión de ticket / check-in** — consume MS1 (`GET /categorias-migratorias`, `POST /tickets`,
  `POST /tickets/{id}/checkin`) y MS2 (`GET /vuelos`)
- Página **Swagger-UI agregada** con los 5 OpenAPI.

Tecnología: React + Vite + cliente `fetch`/axios. El endpoint de API Gateway se configura por variable
de entorno (`VITE_API_BASE`).

### 4.7 Data Science — repo `aeropuerto-data-science`
- **Bucket S3** `s3://<equipo>-aeropuerto-lake/` con prefijos `raw/ms1/`, `raw/ms2/`, `raw/ms3/`.
- **3 contenedores Docker Python** (uno por microservicio con BD), estrategia **pull del 100%**:
  - `ingesta-ms1` → MySQL → CSV por tabla → `raw/ms1/<tabla>/<fecha>/`
  - `ingesta-ms2` → PostgreSQL → CSV por tabla → `raw/ms2/<tabla>/<fecha>/`
  - `ingesta-ms3` → MongoDB → **aplana** `incidencias` → `incidencia`, `incidencia_recurso`,
    `incidencia_vuelo` + `recurso`, `asignacion` → CSV/JSON → `raw/ms3/<coleccion>/<fecha>/`
- **AWS Glue**: base `aeropuerto_lake`, un crawler por prefijo `raw/msX/` → tablas del catálogo.
- **E/R del catálogo**: diagrama que relaciona todas las tablas del catálogo por sus claves de join
  (`vuelo_id`, `pasajero_id`, `recurso_id`, `ruc`, `placa`, `categoria_id`).
- **Athena**: **≥4 consultas** con join entre tablas de ms1 + ms2 + ms3 (las 5 preguntas de §4.5) y
  **≥2 vistas** (`vw_recaudacion_tuua`, `vw_retrasos_hora_punta`). Evidencia en `docs/evidencias/athena/`.
- `docker compose` de los 3 contenedores; ejecución **única** para la carga masiva.

---

## 5. Requerimientos no funcionales

| Requisito | Detalle |
|---|---|
| Despliegue Backend | `docker compose` en **2 VMs de producción** idénticas; **balanceador privado** (ALB/NLB interno); APIs públicas **solo** por **API Gateway con HTTPS**; **3 BDs en una 3ra VM privada** (sin IP pública) |
| Contenedores | Imágenes publicadas en **GHCR** por CI (GitHub Actions) en cada repo de MS; `compose` hace `pull` por tag |
| Reproducibilidad | `user-data` + `docker compose` + `.env.example` + `README` con "cómo levantar tras corte de sesión (<15 min)"; volúmenes de BD respaldados a S3 (dump al cerrar sesión) |
| Datos de prueba | **≥20 000 registros** insertados **una sola vez** en ≥1 tabla de cada BD (`ticket`, `vuelo`, `incidencias`) |
| Documentación de APIs | Swagger-UI navegable de los 5 microservicios + página agregada |
| Diagramas E/R | E/R por cada BD SQL (MS1, MS2) + estructuras JSON documentadas de la NoSQL (MS3) + E/R del catálogo de datos |
| Seguridad | SG mínimos (ver [arquitectura](arquitectura.md)); acceso a instancias por SSM (sin puerto 22 público); credenciales de BD por variables de entorno / `.env` no versionado |
| Observabilidad | Logs de contenedores a CloudWatch (agente) o `docker logs` + `journald`; healthchecks en `compose` |
| Repos públicos | Todos los repos públicos en GitHub; `INDEX.md` con la lista de enlaces |
| Costos | Instancias `t3.small` (`t3.medium` solo VM-DB); apagar VMs fuera de sesiones; 1 sola VM-INGESTA; presupuesto objetivo ≤ $80 |
