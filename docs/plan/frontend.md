# Plan de trabajo — Frontend (Web)

**Rúbrica:** 3 pts · **Hito 1:** avance ≥50% · **Hito 2:** completo.
Documentos base: [requerimientos §4.6](../requerimientos.md) · [arquitectura](../arquitectura.md) ·
[plan general](../plan-de-trabajo.md).

Convención: **d** = días-persona (≈ 3–4 h). Fases: **F0** Mié 2 – Vie 4 · **F1** Sáb 6 – Sáb 12 (Hito 1) ·
**F2** Sáb 13 – Vie 19.

---

## 1. Entregables y Definición de Terminado

- [ ] SPA en **React + Vite** desplegada en **AWS Amplify** (contingencia: S3 static website + CloudFront).
- [ ] **4 vistas** funcionales que consumen **los 5 microservicios**, cada uno con **≥2 métodos REST distintos**.
- [ ] Página **Swagger-UI agregada** con los 5 `openapi.json`.
- [ ] Endpoint base de API Gateway configurable por variable de entorno (`VITE_API_BASE`).
- [ ] Manejo de estados de carga y error en todas las llamadas.
- [ ] Repo público `aeropuerto-frontend` con `README` (levantar local / build) y despliegue automático.

**DoD por vista:** consume su(s) microservicio(s) contra la API real · muestra loading/error/empty ·
navegable desde el menú · sin errores en consola · responsive mínimo (funciona en laptop y móvil).

---

## 2. Asignación

| Trabajo | Responsable | Apoyo |
|---|---|---|
| Toda la SPA (scaffold, 4 vistas, deploy) | **Dev E** | — |
| Gráficos del Dashboard (MS5) | Dev E | Dev D |
| Página Swagger-UI agregada | Dev E | Lead (rutas nginx) |
| CORS en API Gateway / endpoints | Lead | Dev E |

---

## 3. Matriz de cobertura REST (requisito: ≥2 métodos por microservicio)

| MS | Métodos REST invocados por el frontend | Vista(s) |
|---|---|---|
| **MS1** | `GET /categorias-migratorias` · `POST /tickets` · `POST /tickets/{id}/checkin` · `GET /pasajeros/{id}/tickets` | Emisión de ticket / check-in |
| **MS2** | `GET /vuelos` · `GET /vuelos/{id}` · `GET /aeronaves/{placa}/asientos` · `PATCH /vuelos/{id}/estado` | Consulta de vuelo · Emisión de ticket |
| **MS3** | `GET /recursos?estado=Libre` · `POST /incidencias` · `GET /incidencias` · `PATCH /recursos/{id}/estado` | Recursos e incidencias |
| **MS4** | `GET /manifiesto/{vuelo_id}` · `GET /manifiesto/{vuelo_id}/resumen` | Consulta de vuelo + manifiesto |
| **MS5** | `GET /analitica/recursos-mas-fallas` · `/retraso-promedio` · `/incidencias-combustible-por-aerolinea` · `/recaudacion-tuua-por-categoria` · `/vuelos-hora-punta-retrasados` | Dashboard de crisis |

Cada celda cubre ≥2 métodos → requisito satisfecho. Se documenta con capturas del panel Network en el informe.

---

## 4. Tareas

| ID | Tarea | Resp. | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|---|
| FE-01 | Scaffold React + Vite + router + estructura de carpetas | Dev E | 0.5d | F0 | — | `npm run dev` levanta app vacía con menú |
| FE-02 | Cliente HTTP (axios) con `baseURL` por env + interceptores de error/loading | Dev E | 0.5d | F0 | BE-TX-08 | Errores se muestran como toast/banner |
| FE-03 | Pipeline de despliegue en **Amplify** (build en push a `main`) | Dev E | 0.5d | F0 | Learner Lab (R1) | URL pública sirve la app |
| FE-03b | *Contingencia:* bucket S3 static website + CloudFront + script de deploy | Dev E | 0.5d | F0 | si Amplify no está | URL de CloudFront sirve la app |
| FE-04 | Layout base (header, navegación, tema, componentes de loading/error/empty) | Dev E | 0.5d | F0 | FE-01 | Reutilizable por las 4 vistas |
| FE-05 | Mock/wireframe de las 4 vistas (sin datos) para validar con el equipo | Dev E | 0.5d | F0 | FE-04 | Revisado en standup |
| FE-06 | **Vista "Consulta de vuelo + manifiesto"** — `GET /vuelos`, `GET /vuelos/{id}` (MS2) + `GET /manifiesto/{id}`, `/resumen` (MS4) | Dev E | 1.5d | F1 | MS2-05, MS4-02 | Busca vuelo y muestra manifiesto consolidado |
| FE-07 | Deploy de Hito 1: la app en Amplify consume **≥1 MS con ≥2 métodos REST** (MS2) | Dev E | 0.25d | F1 | FE-06 | Evidencia Network con ≥2 requests a MS2 |
| FE-08 | **Vista "Emisión de ticket / check-in"** — `GET /categorias-migratorias`, `POST /tickets`, `POST /tickets/{id}/checkin` (MS1) + `GET /vuelos` (MS2) | Dev E | 1.5d | F2 | MS1-05/07 | Emite ticket y hace check-in end-to-end |
| FE-09 | **Vista "Recursos e incidencias"** — `GET /recursos?estado=`, `POST /incidencias`, `GET /incidencias`, `PATCH /recursos/{id}/estado` (MS3) | Dev E | 1.5d | F2 | MS3-03/05/07 | Lista recursos libres y crea incidencia |
| FE-10 | **Vista "Dashboard de crisis"** — 5 indicadores/gráficos desde MS5 | Dev E + Dev D | 2d | F2 | MS5-03..07 | 5 tarjetas/gráficos con datos reales de Athena |
| FE-11 | Página **Swagger-UI agregada** (lista los 5 `openapi.json` vía selector) | Dev E | 0.5d | F2 | BE-TX-03 | `/docs` agregado navegable |
| FE-12 | Pulido: responsive, estados vacíos, mensajes de error de dependencia caída | Dev E | 1d | F2 | FE-06..10 | Sin errores de consola; funciona en móvil |
| FE-13 | README + capturas para el informe (matriz de cobertura REST) | Dev E | 0.5d | F2 | FE-12 | Sección de informe lista |

**Carga total Dev E ≈ 13 d** en la ventana F0–F2 (≈ 12 días hábiles) → ajustado; el Dashboard (FE-10)
lo comparte con Dev D y el E/R del data lake se le reasigna solo si hay holgura.

---

## 5. Mapa a hitos

**Hito 1 (Sáb 12-Set):** app desplegada en Amplify (o contingencia) + **vista "Consulta de vuelo +
manifiesto"** consumiendo MS2 con ≥2 métodos REST. Suficiente para el "≥50%" de Frontend.

**Hito 2 (Dom 20-Set):** las 4 vistas completas · los 5 MS con ≥2 métodos c/u · Swagger agregado ·
pulido + responsive · capturas para el informe.

---

## 6. Dependencias y camino crítico

```
FE-01..05 (F0, sin backend) ──► FE-06 (necesita MS2-05 + MS4-02) ──► FE-07 deploy Hito 1
                                        │
             FE-08 (MS1)  FE-09 (MS3)  FE-10 (MS5 ← Data Science)  ──► FE-12 pulido ──► FE-13
```

- **FE-06** es el mínimo para Hito 1; solo depende de MS2 y MS4. Si API Gateway no está listo, consume
  MS2 por su URL interna vía un túnel/temporal (documentado).
- **FE-10 (Dashboard)** está en el camino crítico del proyecto (depende de MS5 ← Athena ← ingesta ← 20k).
  Hasta que MS5 responda, se desarrolla contra respuestas de ejemplo.

---

## 7. Riesgos específicos

| Riesgo | Mitigación |
|---|---|
| **Amplify no disponible** en Learner Lab (R1) | Contingencia FE-03b: S3 static website + CloudFront; decidir el Vie 4 |
| **CORS** bloquea llamadas desde el navegador a API Gateway | Configurar CORS en el HTTP API (Lead); probar en F1 |
| **MS5 se atrasa** → Dashboard sin datos | FE-10 contra respuestas mock; conectar en cuanto MS5 esté |
| **Carga de Dev E alta** (~13 d) | Dev D toma los gráficos (FE-10); recortar pulido (FE-12) a lo mínimo si aprieta |
| **URL de API Gateway cambia** tras cada recreación de infra | Leerla de una variable de entorno del build; no hardcodear |
