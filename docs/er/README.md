# Diagramas E/R y esquemas por microservicio

Cada dueño de microservicio deja aquí (o enlaza desde su repo) el modelo de datos de su servicio.

| Archivo | Responsable | Estado |
|---|---|---|
| `ms1-mysql-er.png` / `.drawio` — E/R de MySQL (`persona`, `pasajero`, `categoria_migratoria`, `ticket`, `checkin`, `equipaje`) | Dev A | ⬜ pendiente |
| `ms2-postgres-er.png` / `.drawio` — E/R de PostgreSQL (`vuelo`, `aerolinea`, `aeronave`, `asiento`, `empleado`, `tripulacion`, `operativo_tierra`, `opera_tripulacion`) | Dev B | ⬜ pendiente |
| `ms3-mongo-schemas.md` — JSON Schema de las colecciones `recursos`, `incidencias`, `asignaciones` | Dev C | ⬜ pendiente |
| `er-catalogo-datalake.drawio` — E/R del catálogo de Glue (todas las tablas + claves de join) | Dev D | ⬜ pendiente |

Referencia del modelo relacional monolítico de origen: `Proyecto_LaTeX_Aeropuerto_Jorge_Chavez` (21
relaciones). Ver la transformación aplicada en [`../requerimientos.md`](../requerimientos.md) §3.
