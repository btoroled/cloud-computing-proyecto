# Verificación end-to-end

Cómo probar cada requisito de la rúbrica. Las evidencias (capturas, salidas de consola) se guardan en
`docs/evidencias/` y se incluyen en el informe.

| Qué se prueba | Cómo | Evidencia |
|---|---|---|
| Backend público por HTTPS | `curl https://<api-id>.execute-api.<region>.amazonaws.com/api/pasajeros/health` → `200` | captura + salida |
| ALB privado | `nc -zv <alb-dns> 80` desde fuera de la VPC → **falla**; desde una VM-PROD → OK | 2 capturas |
| BDs privadas | `mysql -h <db-vm> -u ...` desde fuera → **falla**; desde una VM-PROD → OK | 2 capturas |
| Microservicio consume a otro | `POST /api/pasajeros/tickets` con `vuelo_id` inexistente → `422`; el log de MS1 muestra la llamada saliente a MS2 | request + log |
| 1 microservicio sin BD | `ms4-manifiesto-api` no declara ninguna conexión a BD; `GET /api/manifiesto/{id}` responde agregando MS1+MS2+MS3 | código + response |
| 1 microservicio analítico con Athena | `GET /api/analitica/recaudacion-tuua-por-categoria` devuelve el resultado de una query Athena (query id visible en logs / consola Athena) | response + consola Athena |
| ≥20 000 registros | `SELECT COUNT(*) FROM ticket` ≥ 20000 · `SELECT COUNT(*) FROM vuelo` ≥ 20000 · `db.incidencias.countDocuments()` ≥ 20000 | 3 capturas |
| Cada BD SQL con ≥2 tablas relacionadas | Diagrama E/R de MS1 (MySQL) y MS2 (PostgreSQL) en `diagramas/` | 2 diagramas |
| Estructuras JSON de la NoSQL | `docs/er/ms3-mongo-schemas.md` con los JSON Schema de `recursos`, `incidencias`, `asignaciones` | documento |
| Swagger-UI | Abrir `/docs` de cada servicio por la URL pública; la página agregada lista los 5 | 6 capturas |
| Frontend consume los 5 MS | DevTools → Network: ≥2 requests REST distintos por cada uno de los 5 microservicios; app accesible por URL de Amplify/CloudFront | HAR / capturas |
| Data Science — S3 | `aws s3 ls s3://<bucket>/raw/ --recursive` muestra archivos de `ms1/`, `ms2/`, `ms3/` | salida |
| Data Science — 3 contenedores | `docker compose ps` en la VM-INGESTA muestra `ingesta-ms1/2/3` finalizados con éxito; logs con "100% pull" | capturas |
| Data Science — Glue | Consola de Glue: base `aeropuerto_lake` con tablas por archivo | captura |
| Data Science — E/R del catálogo | `diagramas/er-catalogo-datalake.drawio` relacionando todas las tablas del catálogo | diagrama |
| Data Science — Athena ≥4 consultas con join | Ejecutar las 4 consultas en la consola Athena; cada una hace `JOIN` entre ≥2 tablas de ms1/ms2/ms3 | 4 capturas con resultados |
| Data Science — ≥2 vistas | `SHOW VIEWS IN aeropuerto_lake` lista `vw_recaudacion_tuua` y `vw_retrasos_hora_punta` | captura + DDL |
| Diagrama de arquitectura | `diagramas/arquitectura-solucion.drawio` + PNG exportado en el informe, con todos los servicios AWS | diagrama |
| Despliegue en 2 VMs con compose | `docker compose ps` en VM-PROD-1 y VM-PROD-2 con los 5 servicios + nginx `Up` | 2 capturas |

## Prueba de humo E2E (script sugerido)

```
BASE=https://<api-id>.execute-api.<region>.amazonaws.com

curl -s $BASE/api/vuelos/health
curl -s $BASE/api/pasajeros/health
curl -s $BASE/api/infra/health
curl -s $BASE/api/manifiesto/health/dependencias
curl -s $BASE/api/analitica/recursos-mas-fallas?dias=7

# flujo: crear pasajero -> emitir ticket (valida vuelo en MS2) -> check-in -> manifiesto
curl -s -XPOST $BASE/api/pasajeros/pasajeros -d '{...}'
curl -s -XPOST $BASE/api/pasajeros/tickets -d '{"pasajero_id":..., "vuelo_id": 1}'
curl -s -XPOST $BASE/api/pasajeros/tickets/1/checkin -d '{...}'
curl -s $BASE/api/manifiesto/manifiesto/1
```
