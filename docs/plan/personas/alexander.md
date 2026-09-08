# Alexander — Frontend ⚠️

**Repo:** `aeropuerto-frontend` · **Stack:** React + Vite + TypeScript · **Deploy:** AWS Amplify
(contingencia: S3 static website + CloudFront).
**Carga estimada:** ~12.5 d (plan de sección lo estima en ~13 d) — en el límite. Ver [rebalanceo §5](../distribucion-trabajo.md#5-cuellos-de-botella-y-rebalanceo).

Fuentes: [frontend.md](../frontend.md) · [backend.md](../backend.md) §3 (BE-TX-07) ·
[data-science.md](../data-science.md) (DS-13) · [exposiciones-e-informe.md](../exposiciones-e-informe.md).

---

## F0 — Setup (Mié 2 – Vie 4)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | FE-01 | Scaffold React + Vite + router + estructura de carpetas | 0.5d | — | `npm run dev` levanta app vacía con menú |
| ☐ | FE-02 | Cliente HTTP (axios) con `baseURL` por env + interceptores de error/loading | 0.5d | BE-TX-08 | Errores se muestran como toast/banner |
| ☐ | FE-03 | Pipeline de despliegue en **Amplify** (build en push a `main`) | 0.5d | Learner Lab (R1) | URL pública sirve la app |
| ☐ | FE-03b | *Contingencia:* bucket S3 static website + CloudFront + script de deploy | 0.5d | si Amplify no está | URL de CloudFront sirve la app |
| ☐ | FE-04 | Layout base (header, navegación, tema, componentes de loading/error/empty) | 0.5d | FE-01 | Reutilizable por las 4 vistas |
| ☐ | FE-05 | Mock/wireframe de las 4 vistas (sin datos) para validar con el equipo | 0.5d | FE-04 | Revisado en standup |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | FE-06 | **Vista "Consulta de vuelo + manifiesto"** — `GET /vuelos`, `GET /vuelos/{id}` (MS2) + `GET /manifiesto/{id}`, `/resumen` (MS4) | 1.5d | MS2-05, MS4-02 | Busca vuelo y muestra manifiesto consolidado |
| ☐ | FE-07 | Deploy de Hito 1: la app en Amplify consume **≥1 MS con ≥2 métodos REST** (MS2) | 0.25d | FE-06 | Evidencia Network con ≥2 requests a MS2 |

## F2 — Completar, endurecer (Sáb 13 – Vie 19)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | FE-08 | **Vista "Emisión de ticket / check-in"** — `GET /categorias-migratorias`, `POST /tickets`, `POST /tickets/{id}/checkin` (MS1) + `GET /vuelos` (MS2) | 1.5d | MS1-05/07 | Emite ticket y hace check-in end-to-end |
| ☐ | FE-09 | **Vista "Recursos e incidencias"** — `GET /recursos?estado=`, `POST /incidencias`, `GET /incidencias`, `PATCH /recursos/{id}/estado` (MS3) | 1.5d | MS3-03/05/07 | Lista recursos libres y crea incidencia |
| ☐ | FE-10 | **Vista "Dashboard de crisis"** — 5 indicadores/gráficos desde MS5 (Fabricio aporta consultas/formato) | 2d | MS5-03..07 | 5 tarjetas/gráficos con datos reales de Athena |
| ☐ | FE-11 / BE-TX-07 | Página **Swagger-UI agregada** (lista los 5 `openapi.json` vía selector) | 0.5d | BE-TX-03 | `/docs` agregado navegable |
| ☐ | FE-12 | Pulido: responsive, estados vacíos, mensajes de error de dependencia caída | 1d | FE-06..10 | Sin errores de consola; funciona en móvil |
| ☐ | FE-13 | README + capturas para el informe (matriz de cobertura REST) | 0.5d | FE-12 | Sección de informe lista |
| ☐ | DS-13 | *(reasignable)* **E/R del catálogo** `diagramas/er-catalogo-datalake.drawio` — con Fabricio | 1d | DS-09 | Diagrama entregado |
| ☐ | EX-05/06 | Poblar `docs/evidencias/frontend/` + redactar tu sección del informe | 0.5d | verificación | Sección sin "TODO" |

---

## Matriz de cobertura REST (requisito: ≥2 métodos por microservicio)

| MS | Métodos que invoca el frontend | Vista |
|---|---|---|
| MS1 | `GET /categorias-migratorias` · `POST /tickets` · `POST /tickets/{id}/checkin` · `GET /pasajeros/{id}/tickets` | Emisión de ticket / check-in |
| MS2 | `GET /vuelos` · `GET /vuelos/{id}` · `GET /aeronaves/{placa}/asientos` · `PATCH /vuelos/{id}/estado` | Consulta de vuelo · Emisión de ticket |
| MS3 | `GET /recursos?estado=Libre` · `POST /incidencias` · `GET /incidencias` · `PATCH /recursos/{id}/estado` | Recursos e incidencias |
| MS4 | `GET /manifiesto/{vuelo_id}` · `GET /manifiesto/{vuelo_id}/resumen` | Consulta de vuelo + manifiesto |
| MS5 | `recursos-mas-fallas` · `retraso-promedio` · `incidencias-combustible-por-aerolinea` · `recaudacion-tuua-por-categoria` · `vuelos-hora-punta-retrasados` | Dashboard de crisis |

## Entregas por hito

- **Hito 1:** SPA en Amplify consumiendo MS2 con ≥2 métodos REST (vista de vuelo/manifiesto).
- **Hito 2:** 4 vistas completas consumiendo los 5 MS; Swagger agregado; capturas de cobertura REST.

## Informe

- **Sección 5 — Frontend** — capturas de las 4 vistas + panel Network con ≥2 métodos por MS + URL de Amplify.

## Riesgos a tu cargo

- **R1** (con Benja) Amplify no disponible → contingencia FE-03b (S3 + CloudFront); decidir el Vie 4.
- **CORS** bloquea llamadas a API Gateway → Benja configura CORS en el HTTP API; probar en F1.
- **MS5 se atrasa** → FE-10 contra respuestas mock; conectar en cuanto MS5 responda.
- **Sobrecarga (~13 d)** → Fabricio toma FE-10; FE-12 (pulido) es lo primero que se recorta.

## Nota de estado

✅ El repo `aeropuerto-frontend` (PUBLIC) tiene scaffold React+Vite+TS, cliente axios, `amplify.yml`,
Layout, y la vista de vuelos (FE-06 parcial) + placeholders de `tickets` / `infraestructura` /
`dashboard` / `docs`. Falta: FE-08/09/10/11, deploy real verificado, tests, README de "levantar tras
corte". Trabaja hoy con mocks (`VITE_USE_MOCKS`).

## Con todo el equipo

- **BE-TX-01** — acordar `contratos/enums.md` y rangos de ID (F0).
- **EX-04** — exposición virtual con ACL (F1).
- **EX-11** — ensayo general de la demo el Vie 19 (F2).
- **EX-12** — exposición presencial + demo en vivo (F3, **obligatoria**).
