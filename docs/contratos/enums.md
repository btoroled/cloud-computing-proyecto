# Diccionario de enums compartido

Estos valores deben ser **idénticos** (mismo string, misma capitalización, sin tildes donde se indica)
en los 3 microservicios con BD y en los archivos de ingesta, para que los `JOIN` de Athena funcionen.

Cualquier cambio se comunica al equipo en el standup y se versiona aquí.

| Enum | Valores | Usado en |
|---|---|---|
| `estado_vuelo` | `Programado` · `Embarcando` · `Despegado` · `Aterrizado` · `Retrasado` · `Cancelado` | MS2 (`vuelo.estado`), MS4 |
| `tipo_vuelo` | `Nacional` · `Internacional` | MS2 (`vuelo.tipo`), MS5 |
| `tipo_documento` | `DNI` · `Pasaporte` · `Carnet de Extranjeria` | MS1 (`pasajero.tipo_documento`) |
| `alianza` | `Star Alliance` · `SkyTeam` · `Oneworld` | MS2 (`aerolinea.alianza`) |
| `estado_boarding` | `Emitido` · `Check-in` · `Embarcado` · `No-show` · `Cancelado` | MS1 (`ticket.estado_boarding`) |
| `estado_acople` | `Libre` · `Ocupado` · `Mantenimiento` | MS3 (`recursos.manga.estado_acople`) |
| `gravedad` | `Leve` · `Moderada` · `Alta` · `Critica` | MS3 (`incidencias.gravedad`) |
| `tipo_incidencia` | `Falla_Radar` · `Inundacion` · `Falta_Combustible` · `Saturacion_Vial` · `Manga_Inoperativa` · `Otro` | MS3 (`incidencias.tipo_incidencia`), MS5 |
| `tipo_recurso` | `manga` · `radar` | MS3 (`recursos.tipo`) |
| `nombre_categoria_migratoria` | `Nacional` · `Internacional` · `Transito` | MS1 (`categoria_migratoria.nombre`), MS5 |
| `area_operativa` | `Rampa` · `Equipajes` · `Seguridad` · `Mantenimiento` · `Plataforma` | MS2 (`operativo_tierra.area_operativa`) |
| `clase_aeronave` (OACI) | `A` · `B` · `C` · `D` · `E` · `F` | MS2 (`aeronave.clase`, `manga.clase_max` en MS3) |
| `frecuencia_radar` | `Banda L` · `Banda S` · `Banda C` · `Banda X` | MS3 (`recursos.radar.frecuencia`) |

## Rangos de ID (integridad entre servicios)

La data ficticia se genera en orden **MS2 → MS1 → MS3** con un `SEED` fijo (en
`aeropuerto-data-science/seeds/`), para que las FK suaves apunten a registros existentes y los `JOIN`
de Athena no salgan vacíos. Detalle en [plan general §3](../plan-de-trabajo.md#3-datos-de-prueba-integridad-entre-servicios).

| Entidad | Rango de ID | Dueño |
|---|---|---|
| `vuelo.id` | 1 – 25 000 | MS2 |
| `aerolinea.ruc` | RUC de 11 dígitos generado | MS2 |
| `aeronave.placa` | `OB-####` | MS2 |
| `persona.id` / `pasajero.id` | 100 000 – 160 000 | MS1 |
| `ticket.id` | 1 – 60 000 | MS1 |
| `recurso.id` | 1 – 500 | MS3 |
| `incidencia.id` | 1 – 30 000 (referida a `vuelo_id` ∈ [1, 25 000] y/o `recurso.id` ∈ [1, 500]) | MS3 |

Tablas objetivo para el volumen ≥20 000: `ticket` / `equipaje` (MySQL), `vuelo` / `asiento`
(PostgreSQL), `incidencias` (MongoDB).

## Notas

- **Sin tildes** en los valores de enum (`Critica`, `Transito`, `Inundacion`) para evitar problemas de
  encoding en CSV → Glue → Athena. Los textos libres (`descripcion`, `nombre`) sí pueden llevar tildes.
- Fechas y timestamps: **ISO 8601 UTC** (`2026-09-02T14:22:00Z`) en las respuestas de las APIs y en los
  archivos de ingesta.
- Montos (`tarifa`, `precio`): decimal con punto, 2 decimales, en soles (PEN).
- La regla de negocio "una aeronave de clase mayor no puede acoplarse a una manga de clase menor" se
  valida en MS3 usando el orden `A < B < C < D < E < F`.
