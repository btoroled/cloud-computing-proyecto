# Hito 1 — Avance ≥50% (Sáb 12-Set 23:59, exposición virtual ACL)

**Estado real al Jue 10-Set:** MS3 es el único microservicio con código (app + Mongo + `/health` +
modelos + Dockerfile). MS1/MS2/MS4/MS5 son **solo scaffold** (sin `package.json`). AWS: **nada
provisionado todavía**. Frontend: SPA React lista, consume mocks; ya tiene clientes para
`/api/infra/recursos` e `/api/infra/incidencias`. Data Science: solo `seeds/` (generadores Python).

**Tiempo disponible:** viernes 11 + sábado 12 ≈ 2 jornadas, con sesiones de Learner Lab de ~4 h.

Documentos base: [plan general §5](../plan-de-trabajo.md#5-checklist-hito-1-avance-50-por-parte--criterio-del-enunciado) ·
[backend §10](backend.md#10-mapa-a-hitos) · [frontend §5](frontend.md#5-mapa-a-hitos) ·
[data-science §6](data-science.md#6-mapa-a-hitos) · [RUNBOOK](https://github.com/Cloud-MLA/aeropuerto-infra-deploy/blob/main/RUNBOOK.md).

---

## 1. Criterio del enunciado (lo que se evalúa)

| Componente | Mínimo exigido |
|---|---|
| **Backend** | Microservicios implementados parcialmente, **≥1 BD conectada** |
| **Frontend** | Página inicial en **AWS Amplify** que consume **≥1 microservicio con un par de métodos REST** |
| **Data Science** | **VM de ingesta configurada** + **bucket S3 creado** + **≥1 contenedor de ingesta funcionando con datos cargados en S3** |
| **Diagrama de arquitectura** | Diagrama de la solución en **draw.io** (`.drawio` + PNG) con los servicios AWS de esta entrega. Se entrega en Hito 1 con la arquitectura **planificada**; en Hito 2 se ajusta a la infra final ([hito2 §5](hito2.md#5-diagrama-de-arquitectura-de-solución-benja--1-pt)) |

## 2. Estrategia: corte vertical sobre MS3

MS3 cubre los 3 componentes a la vez y es lo más avanzado:

- **Backend** = MS3 con MongoDB conectada + 3 endpoints REST reales.
- **Frontend** = `InfrastructurePage` (ya codificada) apuntando a MS3 real, desplegada en Amplify.
- **Data Science** = 1 contenedor que sube datos de infra generados por `seeds/` a `s3://…/raw/ms3/`.

**Exposición del API para Hito 1:** API Gateway HTTP API con integración **HTTP_PROXY a VM-PROD-1
pública** (sin ALB ni VPC Link — eso es Hito 2).

---

## 3. Tareas urgentes

Convención: **Est.** en horas. Bloque = sesión de trabajo.

### 3.1 Bloque 0 — Jue 10 noche (todos, 30–45 min)

| ID | Tarea | Resp. | Est. | DoD |
|---|---|---|---|---|
| H1-00a | Mensaje al equipo fijando el alcance del Hito 1 (esta tabla) | Benja | 0.25h | Todos confirman su track |
| H1-00b | Cerrar PR #5 (`f0-infra-prep`) → todos parten de `main` actualizado | Benja | 0.25h | `main` al día en `cloud-computing-proyecto` |
| H1-00c | Cada dev deja su repo clonado con dependencias instaladas (`npm install` / imágenes Docker) | Todos | 0.5h | `npm run dev` / `docker compose` arranca en local |

### 3.2 Track A — AWS (Benja)

| ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|
| H1-A1 | Región `us-east-1`; anotar crédito. VPC `aeropuerto` con asistente "VPC and more" (2 AZ, 2 pub, 2 priv, NAT en 1 AZ, endpoint S3) | 0.5h | Learner Lab | VPC + 4 subredes + NAT + endpoint creados |
| H1-A2 | 3 Security Groups: `sg-vm-prod` (in TCP 80 desde 0.0.0.0/0 *temporal*), `sg-vm-db` (3306/5432/27017 desde `sg-vm-prod` y `sg-vm-ingesta`), `sg-vm-ingesta` (sin inbound) | 0.5h | H1-A1 | SGs aplicados |
| H1-A3 | **Bucket S3** `mla-aeropuerto-lake`, block public access ON, prefijos `raw/`, `athena-results/`, `backups/` | 0.25h | — | `aws s3 ls` muestra el bucket → **desbloquea Track D** |
| H1-A4 | 3 EC2 (AL2023, `LabInstanceProfile`, `user-data` Docker del RUNBOOK 1.4): `VM-DB` (t3.medium, privada, `sg-vm-db`), `VM-PROD-1` (t3.small, **pública + IP pública**, `sg-vm-prod`), `VM-INGESTA` (t3.small, privada, `sg-vm-ingesta`) | 0.75h | H1-A2 | SSM abre shell en las 3; `docker --version` OK |
| H1-A5 | `VM-DB`: `docker compose up -d` con `mysql:8` + `postgres:16` + `mongo:7` (RUNBOOK 1.6). Anotar IP privada | 0.5h | H1-A4 | `docker compose ps` → 3 motores `Up` |
| H1-A6 | Publicar al equipo: IP privada VM-DB, IP pública VM-PROD-1, nombre del bucket | 0.1h | H1-A5 | Mensaje enviado |
| H1-A7 | `VM-PROD-1`: `docker compose` con `nginx` (config de infra-deploy) + `ms3` (imagen GHCR). `curl localhost/api/infra/health` | 1h | H1-A5, H1-B7 | nginx enruta `/api/infra` → ms3; health OK |
| H1-A8 | **API Gateway → HTTP API**: integración HTTP URI = `http://<IP-pública-VM-PROD-1>`, route `ANY /{proxy+}`, stage `$default` auto-deploy, **CORS habilitado** | 0.75h | H1-A7 | `curl https://<api-id>.execute-api.us-east-1.amazonaws.com/api/infra/health` responde |
| H1-A9 | Enviar Invoke URL a Alexander (`VITE_API_BASE`) | 0.1h | H1-A8 | URL entregada |
| H1-A10 | Antes del corte: dumps de BD a S3 (RUNBOOK 2.x) + `Stop` instancias | 0.5h | H1-A5 | Backups en `s3://…/backups/` |

### 3.3 Track B — MS3 (Edinson) · rama `feat/recursos-crud`

| ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|
| H1-B1 | `npm install`; correr local con `docker compose up -d mongo_db` + `npm run dev`; `GET /health` → `database: connected` | 0.5h | — | Health reporta Mongo conectada |
| H1-B2 | Corregir `recurso.model.js`: `emun` → `enum` en `tipo` y `clase_max` | 0.1h | H1-B1 | Modelo sin typos |
| H1-B3 | `src/routes/recurso.routes.js`: `GET /recursos` (filtros `?tipo=&estado=`), `POST /recursos` (valida con `recurso.schema.js` + `ajv`), `GET /recursos/:id` | 2h | H1-B2 | 3 métodos responden con `curl` |
| H1-B4 | *(si sobra tiempo)* `PATCH /recursos/:id/estado` (Libre/Ocupado/Mantenimiento) | 0.5h | H1-B3 | Estado inválido → 422 |
| H1-B5 | Montar rutas en `app.js`: `app.use('/api/infra', recursoRoutes)` | 0.1h | H1-B3 | Endpoints bajo `/api/infra` |
| H1-B6 | Seed rápido: script que inserte ~10 recursos (manga/radar) de ejemplo | 0.5h | H1-B3 | `GET /recursos` devuelve datos |
| H1-B7 | Ajustar `.env` / `docker-compose.yml` para `MONGO_URI` → IP privada de VM-DB; tag `v0.1` → verificar imagen en `ghcr.io/cloud-mla/ms3-infraestructura-api:v0.1` | 0.75h | H1-A6, CI | Imagen publicada en GHCR |
| H1-B8 | PR `feat/recursos-crud` → `main` + merge | 0.25h | H1-B3 | PR mergeado |

### 3.4 Track C — Frontend (Alexander)

| ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|
| H1-C1 | `npm ci` + `npm run dev`; confirmar `InfrastructurePage` con mocks | 0.25h | — | App levanta |
| H1-C2 | Alinear shape de datos `src/types/infrastructure.ts` y `src/api/infrastructure.ts` con lo que devuelve MS3 real (`_id`, `tipo`, `nombre`, `estado_acople`) — coordinar con Edinson | 1h | H1-B3 | Tipos coinciden con la respuesta real |
| H1-C3 | **AWS Amplify → Deploy an app → GitHub** → `Cloud-MLA/aeropuerto-frontend`, rama `main` (usa `amplify.yml`) | 0.75h | Learner Lab | URL pública de Amplify sirve la app |
| H1-C4 | Env vars en Amplify: `VITE_USE_MOCKS=false`, `VITE_API_BASE=<Invoke URL API Gateway>`; redeploy | 0.5h | H1-A9, H1-C3 | Build verde con la URL real |
| H1-C5 | Verificar en la URL de Amplify: la página de Infraestructura lista recursos **desde MS3 real** | 0.5h | H1-C4, H1-A8 | Datos reales en pantalla; sin errores de consola |
| H1-C6 | Captura: lista cargada + pestaña Network con la llamada a `execute-api` (≥1 request, idealmente `GET` + `POST`) | 0.25h | H1-C5 | Evidencia guardada en `docs/evidencias/frontend/` |

### 3.5 Track D — Data Science (Fabricio)

| ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|
| H1-D1 | Carpeta `ingesta/ms3/` en `aeropuerto-data-science`: `Dockerfile.python` (plantilla infra-deploy) + script que reutilice `seeds/generators/ms3_infra.py` para generar N filas y escribir CSV/Parquet | 1.5h | — | Contenedor genera archivos localmente |
| H1-D2 | Subida a S3 con `boto3` → `s3://mla-aeropuerto-lake/raw/ms3/incidencias/<fecha>/` (sin credenciales: usa `LabRole` de la VM) | 1h | H1-A3 | `aws s3 cp` de prueba OK contra el bucket real |
| H1-D3 | SSM a `VM-INGESTA` → `docker compose up` del contenedor → verificar objetos en S3 | 0.75h | H1-A4, H1-D2 | `aws s3 ls s3://mla-aeropuerto-lake/raw/ms3/ --recursive` muestra datos |
| H1-D4 | Captura: `aws s3 ls --recursive` con objetos + tamaño | 0.25h | H1-D3 | Evidencia en `docs/evidencias/data-science/` |

### 3.6 Track E — Diagrama de arquitectura en draw.io (Benja)

No depende de que AWS esté arriba: es la arquitectura **planificada** del Hito 1. Se puede hacer el
jueves noche o el viernes en paralelo. Base: el boceto Mermaid v0 en `diagramas/arquitectura-solucion.md`.

| ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|
| H1-E1 | Diagrama en **draw.io** (diagrams.net) con lo que entra al Hito 1: VPC + subredes (2 pub / 2 priv) + SG, IGW/NAT, endpoint S3, `VM-PROD-1` (pública) con nginx + MS3, `VM-DB` (privada) con MySQL/Postgres/Mongo, `VM-INGESTA` (privada), **API Gateway HTTP API** → VM-PROD-1, **S3** `mla-aeropuerto-lake`, **Amplify** (frontend). Marcar en gris/punteado lo que llega en Hito 2 (ALB, VPC Link, VM-PROD-2, Glue, Athena, MS1/2/4/5) | 1.5h | — | Diagrama legible con iconos oficiales de AWS |
| H1-E2 | Exportar a **`diagramas/arquitectura-solucion.drawio`** (editable) + **`diagramas/arquitectura-solucion.png`**; commit + PR a `main` de `cloud-computing-proyecto` | 0.25h | H1-E1 | Ambos archivos versionados; PR mergeado |
| H1-E3 | Incluir el PNG en las slides de la exposición virtual ACL y en la sección de arquitectura del avance | 0.1h | H1-E2 | Slide con el diagrama |

### 3.7 Sáb 12 AM — Integración (Benja coordina)

| ID | Tarea | Est. | DoD |
|---|---|---|---|
| H1-INT1 | Encender EC2 (RUNBOOK Parte 2 pasos 1–4) | 0.25h | Instancias `2/2 checks` |
| H1-INT2 | VM-DB arriba → MS3 en VM-PROD-1 arriba → `curl` local OK | 0.25h | Health por localhost |
| H1-INT3 | API Gateway responde por HTTPS | 0.1h | `curl` a Invoke URL OK |
| H1-INT4 | Amplify build con URL definitiva → frontend consume MS3 | 0.5h | Página con datos reales |
| H1-INT5 | VM-INGESTA corre el contenedor → datos en S3 | 0.25h | Objetos en `raw/ms3/` |
| H1-INT6 | Recolectar todas las evidencias (§4) antes del corte de sesión | 0.5h | Carpeta `docs/evidencias/` completa |

---

## 4. Checklist de entrega (evidencias en `docs/evidencias/`)

- [ ] `GET /health` de MS3 con `database: connected` (captura + link al código de rutas)
- [ ] `curl` a los 3 endpoints REST de MS3 respondiendo (JSON)
- [ ] MongoDB corriendo en VM-DB (`docker compose ps` vía SSM)
- [ ] URL pública de **Amplify** abierta, página de Infraestructura con datos reales + Network tab
- [ ] Consola S3 con `raw/ms3/…` poblado (`aws s3 ls --recursive`)
- [ ] Consola AWS: VPC resource map, lista de EC2 (sin IP pública en VM-DB/VM-INGESTA), bucket con prefijos
- [ ] **Diagrama de arquitectura en draw.io**: `diagramas/arquitectura-solucion.drawio` + `.png` en el repo (H1-E2)
- [ ] PRs mergeados en `ms3-infraestructura-api` y `aeropuerto-frontend`
- [ ] `docs/aws-learner-lab-hallazgos.md` actualizado con estados ✅/⚠️ reales
- [ ] Slides de avance (con el diagrama) + guion de demo corta para la exposición virtual ACL

---

## 5. Camino crítico

```
H1-A1..A5 (VPC, SG, S3, EC2, VM-DB)
        │
        ├──► H1-B7 (MS3 imagen GHCR, MONGO_URI a VM-DB) ──► H1-A7 (MS3 en VM-PROD-1) ──► H1-A8 (API Gateway)
        │                                                                                        │
        │                                                                            H1-C4 (Amplify + URL real) ──► H1-C5/C6 evidencia
        │
        └──► H1-A3 (S3) ──► H1-D2 (boto3→S3) ──► H1-D3 (VM-INGESTA corre) ──► H1-D4 evidencia
```

**Bloqueante #1:** AWS base (H1-A1..A5) — sin esto no avanza ni Backend deploy ni Data Science.
**Bloqueante #2:** API Gateway con HTTPS (H1-A8) — el frontend en Amplify (HTTPS) lo necesita para evitar *mixed content*.

El **diagrama draw.io (Track E)** va **fuera del camino crítico**: es la arquitectura planificada, no
necesita infra desplegada. Hacerlo el jueves noche o el viernes en paralelo.

---

## 6. Si el tiempo se acaba — recortar en este orden

1. **Glue / Athena** → fuera del Hito 1 (no es requisito del 50%).
2. **ALB + VPC Link** → ya recortado (API Gateway directo a VM-PROD pública).
3. **VM-PROD-2** → una sola VM de producción.
4. **`PATCH /recursos/:id/estado` (H1-B4)** → con `GET` lista + `GET` por id + `POST` ya se cumple "un par de métodos REST".
5. **Peor caso de exposición:** si API Gateway pelea, dejar el frontend corriendo en local apuntando a
   `http://<IP-pública-VM-PROD-1>` para la demo y documentar que la URL pública HTTPS queda para Hito 2.

---

## 7. Stretch (solo si hay holgura, no bloquea el Hito 1)

- MS2 (Mariano): scaffold Spring Boot + `GET /vuelos` + `GET /vuelos/{id}/exists` contra Postgres de VM-DB.
- MS1→MS2 o MS3→MS2: una llamada entre servicios con evidencia en logs (adelanta el requisito de Hito 2).
- Segunda VM-PROD + registro en el (futuro) ALB.
- Glue DB `aeropuerto_lake` + 1 crawler sobre `raw/ms3/` + `SELECT count(*)` en Athena.

---

## 8. Checklist personal — Benja

- [ ] Jue 10: kickoff + cerrar PR #5 + fijar decisión de exposición (API Gateway directo)
- [ ] Jue 10 noche / Vie 11 (en paralelo): **diagrama draw.io** H1-E1..E3
- [ ] Vie 11 AM (sesión Lab #1): H1-A1..A6 (VPC, SG, S3, 3×EC2, VM-DB con Mongo, publicar IPs)
- [ ] Vie 11 PM (sesión Lab #2): H1-A7..A9 (nginx+ms3 en VM-PROD-1, API Gateway, entregar URL)
- [ ] Vie 11 noche: H1-A10 (backups + stop)
- [ ] Sáb 12 AM: H1-INT1..INT6 (reinicio, integración E2E, evidencias)
- [ ] Sáb 12: armar paquete de entrega + actualizar `hallazgos.md` + slides ACL (con el diagrama)
