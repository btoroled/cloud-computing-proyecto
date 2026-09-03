# Plan de trabajo — Data Science (Analytics)

**Rúbrica:** 5 pts · **Hito 1:** avance ≥50% · **Hito 2:** completo.
Documentos base: [requerimientos §4.7](../requerimientos.md) · [arquitectura §4](../arquitectura.md) ·
[plan general](../plan-de-trabajo.md).

Convención: **d** = días-persona (≈ 3–4 h). Fases: **F0** Mié 2 – Vie 4 · **F1** Sáb 6 – Sáb 12 (Hito 1) ·
**F2** Sáb 13 – Vie 19.

---

## 1. Entregables y Definición de Terminado

- [ ] **Bucket S3** `s3://<equipo>-aeropuerto-lake/` con prefijos `raw/ms1/`, `raw/ms2/`, `raw/ms3/`.
- [ ] **3 contenedores Docker Python** de ingesta, estrategia **pull del 100%**, uno por microservicio
      con BD; generan CSV/JSON y los cargan a S3.
- [ ] **Catálogo AWS Glue** (`aeropuerto_lake`) con una tabla por archivo cargado.
- [ ] **Diagrama E/R del catálogo** relacionando todas las tablas por sus claves de join.
- [ ] **≥4 consultas Athena** que unen varias tablas (de ms1 + ms2 + ms3) — evidencia con resultados.
- [ ] **≥2 vistas** en Athena (`vw_recaudacion_tuua`, `vw_retrasos_hora_punta`).
- [ ] Repo público `aeropuerto-data-science` con `seeds/`, `ingesta/`, `glue/`, `athena/`, `README`.

**DoD de la ingesta:** el contenedor lee el 100% de las tablas/colecciones de su BD, escribe archivos con
esquema estable a `raw/msX/<tabla>/<fecha>/` y termina con código 0; log indica filas leídas = filas escritas.

**DoD de Athena:** cada consulta corre sin error en la consola, hace `JOIN` entre ≥2 tablas y devuelve
filas; las 2 vistas aparecen en `SHOW VIEWS IN aeropuerto_lake`.

---

## 2. Asignación

| Sub-parte | Responsable | Apoyo |
|---|---|---|
| `seeds/` — generador de datos ficticios compartido | **Dev D** | Dev A/B/C (validan su parte) |
| VM-INGESTA (EC2, `compose`, `LabRole`, red) + bucket S3 | **Lead** | — |
| `ingesta-ms1` (MySQL → CSV → S3) | **Dev A** | Dev D |
| `ingesta-ms2` (PostgreSQL → CSV → S3) | **Dev B** | Dev D |
| `ingesta-ms3` (MongoDB → **aplanado** → CSV/JSON → S3) | **Dev C** | Dev D |
| Catálogo Glue (database + crawlers + esquemas) | **Dev D** | Lead (permisos) |
| Consultas + vistas Athena | **Dev D** | — |
| E/R del catálogo de datos | **Dev D** *(reasignable a Dev E)* | — |

---

## 3. Modelo de archivos en S3 y catálogo

| Prefijo S3 | Origen | Tablas / archivos |
|---|---|---|
| `raw/ms1/` | MySQL (MS1) | `persona`, `pasajero`, `categoria_migratoria`, `ticket`, `checkin`, `equipaje` |
| `raw/ms2/` | PostgreSQL (MS2) | `vuelo`, `aerolinea`, `aeronave`, `asiento`, `empleado`, `tripulacion`, `operativo_tierra`, `opera_tripulacion` |
| `raw/ms3/` | MongoDB (MS3) | `recurso`, `asignacion`, y de `incidencias` **aplanado**: `incidencia`, `incidencia_recurso`, `incidencia_vuelo` |

Claves de join del E/R del catálogo: `vuelo_id`, `pasajero_id` (`persona.id`), `recurso_id`,
`categoria_id`, `aerolinea.ruc`, `aeronave.placa`, `ticket_id`.

---

## 4. Consultas Athena (las 5 que respaldan MS5)

| Q | Endpoint MS5 | Tablas unidas | Lógica | ¿Vista? |
|---|---|---|---|---|
| Q1 | `recursos-mas-fallas?dias=7` | `incidencia` × `incidencia_recurso` × `recurso` | filtrar `fecha_reporte >= hoy-7d`, agrupar por recurso, contar, ordenar desc | — |
| Q2 | `retraso-promedio?tipo=Internacional` | `vuelo` | `avg(date_diff('minute', hora_programada, hora_real))` donde `tipo='Internacional'` y `hora_real` no nulo | — |
| Q3 | `incidencias-combustible-por-aerolinea` | `incidencia` × `incidencia_vuelo` × `vuelo` × `aerolinea` | filtrar `tipo_incidencia='Falta_Combustible'`, agrupar por aerolínea, contar | — |
| Q4 | `recaudacion-tuua-por-categoria` | `ticket` × `pasajero` × `categoria_migratoria` | `sum(tarifa)` por categoría, sobre tickets con `estado_boarding` in (`Check-in`,`Embarcado`) | **`vw_recaudacion_tuua`** |
| Q5 | `vuelos-hora-punta-retrasados` | `vuelo` | `hour(hora_programada)` in (6,7,8,18,19,20); % con `estado='Retrasado'` o retraso > 15 min | **`vw_retrasos_hora_punta`** |

5 consultas con join → cumple el mínimo de 4. 2 vistas → cumple el mínimo de 2.

---

## 5. Tareas

| ID | Tarea | Resp. | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|---|
| DS-01 | Estructura del repo `aeropuerto-data-science` (`seeds/`, `ingesta/`, `glue/`, `athena/`) | Dev D | 0.25d | F0 | — | Repo creado desde plantilla |
| DS-02 | `seeds/` v1: generador con `SEED` fijo, orden **MS2→MS1→MS3**, rangos de ID de [plan general §3](../plan-de-trabajo.md#3-datos-de-prueba-integridad-entre-servicios) | Dev D | 1.5d | F0/F1 | enums | Genera CSV coherentes entre los 3 dominios |
| DS-03 | Escribir las 5 consultas (Q1–Q5) en **SQL sobre Postgres local** para validar la lógica | Dev D | 1d | F0 | modelo | Las 5 devuelven resultados razonables en local |
| DS-04 | **Bucket S3** + estructura de prefijos + política de acceso (`LabRole`) | Lead | 0.25d | F1 | Learner Lab | `aws s3 ls` muestra el bucket |
| DS-05 | **VM-INGESTA** (EC2 `t3.small`) + `docker compose` de los 3 contenedores + acceso lectura a VM-DB | Lead | 0.5d | F1 | BE-INT-02 | `compose` levanta; alcanza la VM-DB |
| DS-06 | **`ingesta-ms1`**: conectar MySQL, `SELECT *` 100% por tabla, escribir CSV, `put_object` a `raw/ms1/` | Dev A | 1d | F1 | DS-04, MS1-02 | Archivos de las 6 tablas en S3 |
| DS-07 | **`ingesta-ms2`**: ídem PostgreSQL → `raw/ms2/` (8 tablas) | Dev B | 1d | F2 | DS-04, MS2-02 | Archivos de las 8 tablas en S3 |
| DS-08 | **`ingesta-ms3`**: leer colecciones + **aplanar `incidencias`** en `incidencia` / `incidencia_recurso` / `incidencia_vuelo` → `raw/ms3/` | Dev C | 1.5d | F2 | DS-04, MS3-02 | 5 archivos en S3; conteos cuadran |
| DS-09 | Glue: database `aeropuerto_lake` + 1 crawler por prefijo `raw/msX/` | Dev D | 0.75d | F2 | DS-06..08 | Tablas visibles en el catálogo |
| DS-10 | Revisión/ajuste de esquemas inferidos por el crawler (tipos, `timestamp`, columnas) | Dev D | 0.75d | F2 | DS-09 | `SELECT * LIMIT 10` correcto por tabla (R9) |
| DS-11 | Implementar Q1–Q5 en **Athena** (workgroup + output S3) | Dev D | 1.5d | F2 | DS-10 | Las 5 corren y devuelven filas |
| DS-12 | Crear las 2 **vistas** `vw_recaudacion_tuua` y `vw_retrasos_hora_punta` | Dev D | 0.5d | F2 | DS-11 | `SHOW VIEWS` las lista; DDL guardado en `athena/` |
| DS-13 | **E/R del catálogo** (`diagramas/er-catalogo-datalake.drawio`) con todas las tablas + claves de join | Dev D / Dev E | 1d | F2 | DS-09 | Diagrama entregado |
| DS-14 | Evidencias en `docs/evidencias/athena/` (capturas de las 5 queries + 2 vistas + `aws s3 ls` + Glue console) | Dev D | 0.5d | F2 | DS-11..13 | Carpeta de evidencias completa |
| DS-15 | Carga masiva real: ejecutar la ingesta **una vez** tras los 20 000 registros del Backend | Dev A/B/C | 0.5d | F2 | Backend F2 (20k) | S3 con el 100% de los datos de las 3 BD |

---

## 6. Mapa a hitos

**Hito 1 (Sáb 12-Set) — criterio del enunciado para Data Science:**
- [ ] **VM de ingesta configurada** (DS-05)
- [ ] **Bucket S3 creado** (DS-04)
- [ ] **≥1 contenedor de ingesta funcionando con datos cargados en S3** (DS-06, `ingesta-ms1`)

**Hito 2 (Dom 20-Set):**
- [ ] Los 3 contenedores corriendo, pull 100% → S3 (DS-06..08, DS-15)
- [ ] Catálogo Glue con tabla por archivo (DS-09/10)
- [ ] E/R del catálogo (DS-13)
- [ ] 5 consultas Athena con join + 2 vistas (DS-11/12)
- [ ] Evidencias (DS-14)

---

## 7. Dependencias y camino crítico

```
seeds/ (DS-02) ──► carga 20k en las 3 BD (Backend F2) ──► ingesta pull 100% (DS-06..08, DS-15)
                                                                   │
                                                                   ▼
                                        S3  ──►  Glue crawlers (DS-09/10)  ──►  Athena Q1–Q5 + vistas (DS-11/12)
                                                                                        │
                                                                                        ▼
                                                                         MS5 (Backend §8)  ──►  Dashboard Frontend (FE-10)
```

**Esta cadena es el camino crítico de todo el proyecto.** Cualquier atraso en `seeds/`, en la carga de
20k o en la ingesta empuja MS5 y el Dashboard. Por eso `seeds/` (DS-02) y `ingesta-ms1` (DS-06) se
adelantan a F1 aunque el resto de la ingesta sea F2.

---

## 8. Riesgos específicos

| Riesgo | Mitigación |
|---|---|
| **Glue infiere tipos/columnas mal** desde CSV (R9) | `CREATE TABLE` explícito en `glue/` o exportar en **Parquet** con esquema; validar con `SELECT ... LIMIT 10` (DS-10) |
| **Costo de Athena** por escaneo completo (R4) | Particionar por `<fecha>` en el prefijo; en pruebas usar `LIMIT`; formato columnar si da tiempo |
| **IDs no cruzan** entre dominios → joins vacíos (R5) | `seeds/` con orden MS2→MS1→MS3 y `SEED` fijo; DS-03 valida los joins en local antes de Athena |
| **`incidencias` anidado** rompe el catálogo | Aplanado obligatorio en `ingesta-ms3` (DS-08) a 3 archivos planos |
| **Depende de los 20k del Backend** | Si el Backend se atrasa, ingestar lo que haya y re-ejecutar (DS-15); dejar constancia en el informe |
| **`LabRole` sin permisos para Glue/Athena** | Validar en F0 ([hallazgos Learner Lab](../aws-learner-lab-hallazgos.md)) |
