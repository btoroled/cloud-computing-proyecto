# Benja — Líder / Arquitecto

**Repos que lidera:** `aeropuerto-infra-deploy`, `cloud-computing-proyecto` (docs)
**Carga estimada:** ~15 d · **Foco:** contratos + CI, infra de red, despliegue, API Gateway, diagrama,
consolidación de informe/PPT.

Fuentes: [backend.md](../backend.md) §3 y §9 · [data-science.md](../data-science.md) §5 ·
[diagrama-arquitectura.md](../diagrama-arquitectura.md) · [exposiciones-e-informe.md](../exposiciones-e-informe.md) ·
[plan general](../plan-de-trabajo.md).

---

## F0 — Setup y contratos (Mié 2 – Vie 4)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ✅ | — | Crear la organización GitHub + los 9 repos (README + LICENSE + plantilla) e invitar al equipo | 0.5d | — | 9 repos creados; equipo con acceso |
| ⛔ | — | ~~Validar Learner Lab en orden EC2+VPC → S3 → Glue+Athena → API Gateway+VPC Link → Amplify y escribir `aws-learner-lab-hallazgos.md`~~ | 0.5d | Learner Lab | **Descartada (2026-09-22, decisión de Benja)** — el despliegue real ya está hecho y verificado (ver registro de avance); no se justifica retroceder a documentar el hallazgo formal en la cuenta de Jobeth |
| 🟡 | BE-TX-01 | Acordar y versionar `contratos/enums.md` y rangos de ID (con todo el equipo) | 0.5d | — | Archivo aprobado en repo docs |
| ✅ | BE-TX-02 | Plantilla de repo de microservicio (estructura, `.editorconfig`, `.env.example`, `README` base) | 0.5d | — | 5 repos creados desde plantilla |
| ✅ | BE-TX-04 | Imagen base Docker por lenguaje (Py 3.12-slim, Temurin 21, Node 20-alpine) + healthcheck | 0.5d | — | `plantilla/docker/Dockerfile.{python,java,node}` con `HEALTHCHECK`; las 5 imágenes reales ya construyen y publican (a Docker Hub, no GHCR) |
| ✅ | BE-TX-08 | Contrato de errores común (JSON de error, códigos 400/404/422/502) | 0.25d | — | Documentado en `contratos/` |
| ✅ | BE-INT-01 | VPC + subredes privadas + IGW/NAT + Security Groups | 1d | Learner Lab OK | SGs mínimos aplicados |
| ✅ | BE-INT-02 | 4 EC2: VM-PROD-1, VM-PROD-2, VM-DB, VM-INGESTA + acceso por SSM (sin puerto 22 público) | 0.5d | BE-INT-01 | Acceso por SSM funcionando |
| ✅ | DA-01 | Boceto v0 del diagrama con la topología planeada | 0.5d | [arquitectura](../../arquitectura.md) | `diagramas/arquitectura-solucion.md` (Mermaid) — superado por DA-03 (v2 final) |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ✅ | BE-TX-05 | GitHub Actions: build + push a Docker Hub en push a `main`/tag `vX.Y` | 0.5d | BE-TX-02 | Tag/push dispara imagen `btoroled/<repo>:<tag>` (migrado de GHCR el 2026-09-22) |
| ✅ | BE-TX-06 | `nginx.conf` reverse proxy por path (`/api/pasajeros`, `/api/vuelos`, …) | 0.5d | contratos | nginx enruta a los 5 servicios — verificado con `curl` real |
| ✅ | BE-INT-03 | VM-DB: `compose` con `mysql:8` + `postgres:16` + `mongo:7` + volúmenes | 0.5d | BE-INT-02 | Los 3 motores `Up`/`healthy` — verificado por SSM |
| ✅ | BE-INT-04 | VM-PROD ×2: `compose` con nginx + MS1..MS5 (pull de Docker Hub) | 1d | BE-TX-06, imágenes | 5 servicios `Up` en ambas VMs — verificado por SSM |
| ✅ | BE-INT-05 | ALB interno (o NLB) → target de las 2 VM-PROD:80 | 0.5d | BE-INT-04 | Balancea entre las 2 VMs — 2/2 targets `healthy` |
| ✅ | BE-INT-06 | API Gateway HTTP API + VPC Link → ALB; ruta `/{proxy+}` | 1d | BE-INT-05, Learner Lab | URL pública HTTPS responde `/api/*/health` — verificado con `curl` desde afuera (200) |
| 🟡 | BE-INT-07 | `RUNBOOK.md` de reinicio tras corte (< 15 min) + script de dumps a S3 | 0.5d | BE-INT-03 | Script de dumps probado con datos reales (3.6MB MySQL + 14KB Postgres + 2KB Mongo en S3); **falta** cronometrar un reinicio real tras corte |
| ✅ | DS-04 | Bucket S3 + estructura de prefijos `raw/ms{1,2,3}/` + política `LabRole` | 0.25d | Learner Lab | `aws s3 ls` muestra el bucket — creado 2026-09-22 |
| 🟡 | DS-05 | VM-INGESTA (EC2 `t3.small`) + `compose` de los 3 contenedores + acceso lectura a VM-DB | 0.5d | BE-INT-02 | EC2 lista y alcanza VM-DB; **compose sigue siendo el placeholder** — falta que Data Science traiga los 3 reales |
| ⛔ | DA-02 | ~~Diagrama v1 tras validar Learner Lab~~ | 0.5d | hallazgos Learner Lab | **Saltado** — se fue directo a DA-03 (v2 final) con la infra real ya desplegada, sin pasar por un v1 intermedio |
| 🟡 | EX-01 | Plantilla del informe (Word/Docs) con las 9 secciones + placeholders de evidencia | 0.5d | — | Plantilla compartida |
| ✅ | EX-02 | Slides de avance Hito 1 (1–2 por integrante) + consolidación | 1d | avance F1 | PPT de avance listo (`ppt/avance-hito1.pptx` existe en el repo) |
| 🟡 | EX-03 | Guion de la demo corta de Hito 1 + ensayo interno | 0.5d | deploy v1 | Demo de ≤10 min ensayada |
| ☐ | EX-04 | **Exposición virtual con ACL** (con todo el equipo) | 0.5d | EX-02/03 | Realizada; feedback del ACL anotado |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ✅ | BE-INT-08 | Prueba de humo E2E por la URL pública + verificación de "privado" (ALB/BD inaccesibles) | 0.5d | todo F2 | Script E2E verde — `docs/evidencias/infra/smoke-e2e-2026-09-22.txt` |
| ✅ | BE-INT-09 | Logs de contenedores a CloudWatch (o local) + healthchecks en `compose` | 0.5d | BE-INT-04 | Logs locales (`json-file`, ya cumplía la alternativa "o local"); healthchecks agregados y verificados `healthy` en los 6 servicios de ambas VM-PROD |
| 🟡 | DA-03 | Diagrama v2 final: nombres reales de recursos, todos los servicios, IDs de subred/SG | 0.5d | despliegue F2 | Coincide 1:1 con lo desplegado |
| ☐ | DA-04 | Revisión cruzada del diagrama con Fabricio (DS) y Alexander (Frontend) | 0.25d | DA-03 | Sin observaciones pendientes |
| ☐ | EX-07 | PPT resumen del Hito 2 (estructura + 2–3 slides por integrante) | 1d | EX-06 | PPT completo |
| ☐ | EX-11 | Ensayo general de la demo en vivo (Vie 19) + reparto de quién muestra qué (con todos) | 0.5d | code freeze | Demo de ~15 min cronometrada |

## F3 — Entrega y exposición (Sáb 20 / Semana 7)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| 🟡 | DA-05 | Export PNG del diagrama + insertar en informe y PPT | 0.25d | DA-04 | PNG en `informe/` y `ppt/` |
| ☐ | EX-08 | Consolidación final del informe + revisión cruzada + export **PDF** | 1d | EX-06 | PDF final revisado por ≥2 personas |
| ✅ | EX-09 | Verificar `INDEX.md` (todos los repos públicos y accesibles) | 0.25d | repos | Enlaces abren sin login |
| ☐ | EX-10 | **Subida a Canvas** (informe PDF + PPT + enlaces) antes del Mar 23-Set 23:59 (aplazado) | 0.25d | EX-07/08/09 | Entrega confirmada |
| ☐ | EX-12 | **Exposición presencial + demo** (con todos) | 0.5d | EX-11 | Realizada |

---

## Secciones del informe a tu cargo

- **1. Introducción y objetivos**
- **2. Arquitectura de solución** — diagrama `draw.io` + lista de servicios AWS
- **7. Despliegue y seguridad** — `docker compose ps` ×2 · `nc`/`mysql` fallando desde fuera · URL HTTPS respondiendo · `RUNBOOK.md`
- **8. Enlaces a repositorios** — `INDEX.md`
- **9. Conclusiones y limitaciones** — contingencias aplicadas (Amplify, VPC Link, etc.)

## Riesgos a tu cargo

- **R2** VPC Link / API Gateway privado no funciona con `LabRole` → contingencia NLB o ALB público solo para demo.
- **R3** Temporizador de sesión (~4 h) apaga EC2 → todo reproducible (`user-data` + `compose` + GHCR); `RUNBOOK.md`; dumps a S3.
- **R4** Presupuesto (~$50–80) → `t3.small`/`micro`; apagar VMs fuera de sesión; Athena particionado.
- **R1** (con Alexander) Amplify no disponible → S3 static website + CloudFront.
- **R7** (con Mariano) Java pesado en `t3.small` → `-Xmx256m` + límite de memoria; si no, `t3.medium`.

## Bloqueos que generas para otros

- **BE-INT-06 (API Gateway + VPC Link)** habilita el consumo real de todos → validar viabilidad en F0, cerrar en F1.
- **DS-04/DS-05 (S3 + VM-INGESTA)** habilitan toda la ingesta de Data Science.
- **contratos/enums (BE-TX-01)** son prerequisito de todos los microservicios.

---

## Registro de avance

### ✅ Crear org + 9 repos + invitar al equipo (2026-09-08)

- Organización `Cloud-MLA` creada; **6 miembros** unidos (btoroled, Guillermo-Heredia, MarianoUtec,
  Edinson695, AIexander-lx, cruz-eng) — sin invitaciones pendientes.
- **9 repos** (8 en `Cloud-MLA` + `cloud-computing-proyecto` en `btoroled/`, se queda ahí):
  - Ya existían: `aeropuerto-frontend`, `aeropuerto-data-science`, `ms3-infraestructura-api`,
    `ms4-manifiesto-api`, `ms5-analitica-api` — todos **públicos**.
  - Creados ahora (públicos, README + `.gitignore` + MIT): `ms1-pasajeros-api`, `ms2-vuelos-api`,
    `aeropuerto-infra-deploy`.
- **Pendiente menor:** `aeropuerto-frontend` no tiene archivo `LICENSE` (los demás sí, MIT) — agregarlo
  por PR normal (el intento por API quedó bloqueado).

### 🟡 Validar Learner Lab (2026-09-08)

- `docs/aws-learner-lab-hallazgos.md` convertido en checklist accionable (6 bloques con "cómo probar" +
  columnas de resultado).
- `aeropuerto-infra-deploy/RUNBOOK.md` v1: provisión inicial paso a paso, reinicio tras corte, limpieza.
- **Pendiente:** ejecutar la validación en la consola del Learner Lab y rellenar la tabla.

### ✅ BE-TX-08 — Contrato de errores (2026-09-08)

- `docs/contratos/errores.md`: formato JSON único, mapa de códigos HTTP (400/404/409/422/502/503/500),
  catálogo de `error.code` (comunes + de dominio), propagación con `X-Request-Id`, notas por stack.

### 🟡 BE-TX-01 — enums + rangos de ID (2026-09-08)

- `docs/contratos/enums.md` ya tenía el diccionario de enums; le agregué la sección **Rangos de ID**
  (dueño por entidad) consolidando el plan §3.
- **Pendiente:** revisión y OK del equipo en el standup.

### 🟡 BE-TX-02 / BE-TX-04 / BE-TX-05 / BE-TX-06 — plantilla + imágenes base + CI + nginx (2026-09-08)

- `aeropuerto-infra-deploy/plantilla/`: `.editorconfig`, `.env.example`, `.gitignore`, `README.servicio.md`,
  `docker/Dockerfile.{python,java,node}` (healthcheck + no-root; MS2 con `-Xmx256m`),
  `github/workflows/build-push-ghcr.yml` (push de tag `vX.Y` → `ghcr.io/cloud-mla/<repo>`).
- `aeropuerto-infra-deploy/nginx/nginx.conf`: reverse proxy por path (BE-TX-06).
- PR: `Cloud-MLA/aeropuerto-infra-deploy#1`.

### ✅ BE-TX-02 — archivos base aplicados a los repos de MS (2026-09-08)

- `ms1` / `ms2` / `ms4` / `ms5`: PR `chore/plantilla-base` con `.editorconfig`, `.gitignore` (ms1/ms2),
  `.env.example` por stack, `.github/workflows/build-push-ghcr.yml`, README con convenciones + checklist
  de scaffold. **Sin código de app** (eso es `MSx-01` de cada dev).
- `ms3` mantiene su propio scaffold (Edinson ya avanzó MS3-01).
- **Pendiente:** que cada dev haga su scaffold (`MSx-01`) y corra el primer `git tag` para publicar imagen.

### 🟡 BE-INT-03 / BE-INT-04 — compose de VM-DB y VM-PROD (2026-09-08)

- `aeropuerto-infra-deploy/compose/vm-db/` (mysql 8 + postgres 16 + mongo 7, healthchecks, volúmenes).
- `aeropuerto-infra-deploy/compose/vm-prod/` (nginx + MS1..MS5 desde GHCR, `.env.example`).
- **Pendiente:** desplegarlos en las EC2 cuando existan (depende de BE-INT-02) y de que haya imágenes en GHCR.

### 🟡 DA-01 — boceto v0 del diagrama (2026-09-08)

- `diagramas/arquitectura-solucion.md` — diagrama Mermaid con la topología completa (VPC, subredes,
  API Gateway+VPC Link, ALB, VM-PROD/DB/INGESTA, S3/Glue/Athena, Amplify, SSM/CloudWatch, GHCR).
- **Pendiente:** `.drawio` + PNG con nombres reales para el informe (DA-03, F2).
- Renderiza directo en GitHub; sirve para la exposición virtual del Hito 1.

### 🟡 EX-01 / EX-03 — plantilla del informe + guion de demo (2026-09-08)

- `informe/plantilla-informe.md` — 9 secciones con responsables, placeholders de evidencia `[E-n]` y
  anexo de trazabilidad rúbrica → evidencia.
- `informe/guion-demo.md` — guion para Hito 1 (≤10 min) y presencial (~15 min) + checklist previo.
- **Pendiente:** llenar las secciones en F2 (EX-06) y ensayar la demo (EX-11).

### 🟡 BE-INT-07 / BE-INT-08 — scripts de operación (2026-09-08)

- `aeropuerto-infra-deploy/scripts/`: `user-data-*.sh`, `provision.sh` (aws-cli), `smoke-e2e.sh`
  (prueba de humo E2E), `backup-db.sh` / `restore-db.sh`.
- **Pendiente:** correr `smoke-e2e.sh` contra la URL real (depende de BE-INT-06).

### 🔴 Auditoría de avance real (2026-09-21, día de entrega Hito 2)

Verificado contra el estado real de los 9 repos (no solo contra este checklist), sin sesión AWS activa:

- ✅ **EX-09** — `INDEX.md` completado con las 9 URLs reales; confirmado que los 9 repos son
  públicos (`gh api .../visibility` → `public` en los 9). Se corrigió además una inconsistencia:
  el doc decía `ghcr.io/btoroled/<repo>`, el registro real que usan los 5 workflows es
  `ghcr.io/cloud-mla/<repo>`.
- 🔴 **Hallazgo crítico — registro de imágenes inconsistente:** `RUNBOOK.md` y
  `compose/vm-prod/docker-compose.yml` asumen **Docker Hub público** (`btoroled/<repo>`), pero
  ninguna imagen existe ahí (verificado con la API pública de Docker Hub, las 5 devuelven 404). Los
  5 microservicios (`ms1`..`ms5`) todavía tienen el workflow `build-push-ghcr.yml` sin migrar —
  publican a **GHCR**, no a Docker Hub. Hay un cambio a medias sin commitear en este repo
  (`plantilla/github/workflows/build-push-ghcr.yml` → `build-push-dockerhub.yml`) que nunca se
  aplicó a los 5 repos de MS. **Riesgo real: `docker compose pull` en VM-PROD puede fallar.**
  Pendiente decidir un solo registro y propagarlo a los 5 repos + RUNBOOK + compose.
- 🔴 Solo **MS4 y MS5** (Fabricio) tienen tag `v1.0`; MS1/MS2/MS3 no tienen ningún tag todavía
  (solo push a `main`, que sí dispara `:latest` por el workflow).
- 🔴 **DA-03/DA-04/DA-05** — no existe ningún `.drawio` ni PNG en `diagramas/`, solo el boceto
  Mermaid (`arquitectura-solucion.md`). Diagrama final pendiente de verdad.
- 🔴 `docs/evidencias/` prácticamente vacío (solo `README.md` y un `.gitkeep` en `athena/`).
- 🔴 `informe/informe.tex` tiene **25 evidencias/figuras sin completar** (`\evidencia{...}` ×20,
  `\figuraPendiente{...}` ×5) repartidas en las secciones de **todo el equipo**, no solo las mías —
  necesita capturas/salidas reales de cada quien hoy mismo.
- 🔴 No existe PPT de Hito 2 en `ppt/` (solo `avance-hito1.pptx`).
- **No pude verificar infra en vivo** (VPC/EC2/ALB/API Gateway/S3) — token de AWS inválido, sesión
  del Learner Lab no estaba activa al momento de esta auditoría.
- 🔴 **VM-PROD pública por decisión temporal:** `terraform/ec2.tf` + `security_groups.tf` (editados
  hoy) ponen las 2 VM-PROD en subred pública con IP pública y `sg-vm-prod` abierto a `0.0.0.0/0` en
  80 y 8001–8005, para poder probar sin ALB/API Gateway mientras el registro de imágenes estaba roto.
  **Decisión (2026-09-21):** se despliega así hoy para avanzar rápido; **hay que cerrar `sg-vm-prod`
  (dejar solo `sg-alb`) y volver a subred privada antes de capturar la evidencia final** — si no, el
  DoD de BE-INT-06 y el ítem "APIs públicas solo por API Gateway" del checklist quedan sin cumplir.
  **No olvidar este paso al final.**

### 🟡 DA-03 / DA-05 — diagrama de arquitectura v2 (2026-09-21)

- Generado `diagramas/arquitectura-solucion.drawio` (editable, mxGraph/draw.io) y
  `diagramas/arquitectura-solucion-v2.png` (export), con los nombres reales de recursos tomados de
  `aeropuerto-infra-deploy/terraform/` (VPC `10.0.0.0/16`, 4 subredes, 5 SG, 4 EC2, ALB, VPC Link, API
  Gateway) — no de un boceto a mano.
- Incluye los 9 flujos pedidos por el DoD: carga del SPA, REST del browser a API Gateway, VPC Link → ALB
  → VM-PROD×2, MS1→MS2/MS3→MS2/MS4→MS1‑2‑3 (consumo entre servicios), MS-DB por los 3 motores,
  VM-INGESTA → VM-DB y → S3, S3 → Glue → Athena, MS5 → Athena (boto3), VM-PROD → GHCR vía NAT.
- **Ya insertado en `informe/informe.tex` §Arquitectura de solución** (reemplacé el
  `\figuraPendiente` por un `\includegraphics` real).
- Dibuja el **estado objetivo** (VM-PROD privada, ver nota arriba sobre `sg-vm-prod` temporalmente
  público) — si la infra real desplegada hoy termina distinta, hay que regenerarlo.
- **Pendiente real:** DA-04 (revisión cruzada con Fabricio/Alexander) y confirmar contra la consola AWS
  una vez desplegado; falta insertarlo también en el PPT del Hito 2 (EX-07, todavía no existe).

### 🟢 ingesta-ms1 / ingesta-ms2 desplegados y corridos en VM-INGESTA (2026-09-23)

- Empaqueté el código de Guillermo (`ingesta-ms1`) y Mariano (`ingesta-ms2`) desde
  `aeropuerto-data-science/ingesta/`, lo subí a S3 y lo desplegué en VM-INGESTA (reemplazando mi
  placeholder). Construí las 2 imágenes reales con `docker compose build` y las corrí de verdad
  (`docker compose run --rm`), apuntando a la VM-DB real con las credenciales del `terraform.tfvars`.
- **Pipeline confirmado funcional de punta a punta:** conecta a MySQL/PostgreSQL reales, extrae las
  tablas, sube a `s3://mla-aeropuerto-lake/raw/ms{1,2}/`. Evidencia: `raw/ms2/*/2026-09-23/*.csv` en S3.
- **0 filas porque la VM-DB de hoy está vacía** — Guillermo/Mariano ya habían cargado 20k contra sus
  propios entornos (locales o la cuenta AWS vieja de Benja), pero no contra esta infra nueva. **Falta
  que ellos recarguen sus seeds contra `DB_HOST=10.0.10.170`** (o me pasan el comando y lo corro yo).
- `ingesta-ms3` sigue sin existir (bloqueado en Edinson) — es el único de los 3 que falta.
- Nota menor: el script de Guillermo salta el upload cuando una tabla tiene 0 filas; el de Mariano sube
  igual un CSV vacío (solo encabezado). No es un bug, solo una diferencia de criterio entre ambos.

### 🟢 Cierre de la cadena de datos: carga masiva + Glue + Athena (2026-09-23)

- **Carga masiva real contra la VM-DB de hoy:** MS1 (`load_csv.py` de Guillermo) → 184 394 filas,
  **25 000 tickets**. MS2 (generador de Mariano) → **20 500 vuelos** + 21 000 asientos. Ambos
  verificados con `COUNT(*)` independiente, no solo el log del script
  (`docs/evidencias/backend/ms{1,2}-count-2026-09-23.txt`).
- Re-corrí `ingesta-ms1`/`ingesta-ms2` sobre estos datos reales → `s3://mla-aeropuerto-lake/raw/ms{1,2}/`
  ya tiene el 100% real, no CSVs vacíos.
- **DS-09 (Glue):** creé `aeropuerto_lake` + 1 crawler por prefijo (`LabRole`). 14 tablas catalogadas
  (6 de MS1, 8 de MS2); el crawler de MS3 corrió pero no encontró nada (bloqueado en Edinson).
- **DS-10 (esquemas):** encontré y arreglé un bug real — el crawler infiere `hora_programada`/
  `hora_real` como `varchar` (formato real `2026-09-30 06:15:00+00:00`, ISO8601 con offset), y las
  queries de Fabricio que usaban `date_diff()`/`EXTRACT()` directo fallaban. Arreglado envolviendo con
  `from_iso8601_timestamp(replace(col,' ','T'))` en vez de forzar el tipo de columna en el catálogo.
- **DS-11 (Athena):** configuré el workgroup (`OutputLocation=s3://mla-aeropuerto-lake/athena-results/`)
  y corrí las 5 queries reales de Fabricio. **3 de 5 (Q2, Q4, Q5) funcionan con datos reales** — evidencia
  completa en `docs/evidencias/athena/`. Q1 y Q3 necesitan la tabla `incidencia` (MS3), bloqueadas.
- **DS-12 (vistas):** las 2 vistas (`vw_recaudacion_tuua`, `vw_retrasos_hora_punta`) creadas y
  confirmadas con `SHOW VIEWS` — la segunda necesitó el mismo fix de timestamps que Q5.
- Commits en `aeropuerto-data-science` (fix de las 3 queries) y en `cloud-computing-proyecto`
  (checklist + evidencia), todo pusheado.

### 🟢 EX-07 — PPT resumen del Hito 2 (2026-09-23)

- Generé `ppt/hito2-resumen.pptx` (18 slides) con `python-pptx`: portada, agenda, y las 9 secciones del
  plan (intro, arquitectura + diagrama, backend, transformaciones, frontend, data science, despliegue,
  repos, conclusiones), con el estado real de hoy (no aspiracional) y la evidencia recolectada en esta
  sesión. El equipo puede seguir editándolo directamente.

### 🟢 ingesta-ms3 construido y las 5 queries de Athena ya funcionan (2026-09-23)

- Ante la falta de tiempo, construí `ingesta-ms3` yo mismo adaptando el aplanado de referencia que
  **ya había escrito Fabricio** (`athena/local-postgres/aplanar_ms3.py`, explícitamente marcado como
  "esto lo hará ingesta-ms3 en F2") — no fue inventar lógica de negocio nueva, fue conectar 2 piezas
  que ya existían: su generador de seed (`seeds/generators/ms3_infra.py`) + su aplanado, adaptado para
  leer de una MongoDB real en vez de JSONL local.
- Cargado en MongoDB real: 500 recursos, **25 000 incidencias**, 40 000 asignaciones (`mongoimport`,
  verificado con `countDocuments()` independiente).
- `ingesta-ms3` corrido de verdad: 5 archivos en `s3://mla-aeropuerto-lake/raw/ms3/` con datos reales.
- Encontré y arreglé un bug real en el camino (DS-10): el SerDe simple de Glue no soporta comillas CSV
  — una coma dentro del texto libre de `descripcion` corrompía las columnas siguientes. Arreglado
  saneando la coma en el origen, no cambiando el SerDe (para no romper el tipado del resto).
- **Resultado: las 5 consultas de Athena (Q1-Q5) y las 2 vistas ya funcionan con datos reales de las
  3 fuentes.** Catálogo Glue con 19 tablas. Evidencia completa en `docs/evidencias/athena/`.
- Avisarle a Edinson: su trabajo de generador/aplanado era correcto y se usó tal cual — solo faltaba
  conectarlo a una MongoDB real, que es justo el paso donde estaba trabado por tiempo.

### 🔴 Revisión de riesgos (2026-09-23) — qué puede salir mal antes de la entrega/demo

- **`terraform/ec2.tf` y `variables.tf` NO reflejan lo realmente desplegado.** Tienen 161+56 líneas de
  cambios sin commitear desde antes de esta sesión (no son míos). Si alguien clona el repo y corre
  `terraform apply` desde cero, **no reproduce la infra actual** — riesgo real para el punto de la
  rúbrica de "infra reproducible por Terraform". Pendiente: revisar y commitear esos 2 archivos.
- **El `.tfstate` de Terraform solo existe en mi máquina** (correctamente en `.gitignore`, pero no hay
  backend remoto). Si otra persona necesita tocar la infra por Terraform, no tiene el state — quedaría
  desincronizado o intentaría recrear recursos que ya existen. Considerar backend S3 si el equipo va a
  seguir iterando la infra.
- **VM-PROD sigue con IP pública asignada** (aunque bloqueada por `sg-vm-prod`). Si alguien revierte el
  commit de hoy o reabre el SG por error, vuelve a quedar expuesta al toque. Un grader que mire la
  consola EC2 (no solo pruebe conectividad) podría notar la IP pública igual, aunque inalcanzable.
- ✅ **Swagger arreglado en las 5 APIs** — MS1/MS2 ya funcionaban; arreglé el ruteo de nginx para MS4/MS5
  (no le recortaba el prefijo, mismo patrón que el actuator de MS2); y a **MS3 le agregué Swagger desde
  cero** (no existía en el código — spec OpenAPI 3.0 escrito a mano con `swagger-ui-express`, a partir
  de las rutas y schemas Ajv reales). Los 5 verificados 200 vía gateway el 23-Set. Item "Swagger-UI
  navegable de las 5 APIs" cerrado; falta solo la página agregada (de Alexander).
- 🟡 **`terraform/ec2.tf` y `variables.tf` ya commiteados** (2026-09-23) — ya no hay drift entre lo
  desplegado y el repo. Sigue pendiente: backend remoto del `.tfstate` si el equipo va a seguir
  iterando la infra desde otra máquina.
- 🟢 **Script de reactivación creado y probado:** `scripts/reactivar-sesion.sh` automatiza el RUNBOOK
  Parte 2 (arranca las 4 EC2, levanta `docker compose` en VM-DB/VM-PROD, espera el ALB, prueba API
  Gateway). Corrido hoy en modo `--check`: 4/4 instancias, 2/2 targets healthy, API Gateway 200.
- **Evidencia de consumo MS1→MS2 y MS4→MS1/2/3 no capturada explícitamente** (solo MS3→MS2 tiene log
  guardado) — el código y los endpoints existen, falta la captura antes del PDF final.
- **Reinicio tras corte de sesión sigue sin cronometrarse** — el RUNBOOK lo documenta pero nunca se
  probó un corte real con la infra de hoy. Riesgo para la exposición de Semana 7 si el Lab se corta
  a mitad de la demo y nadie sabe cuánto tarda en verdad volver a levantar todo.
- **La sesión de AWS de Jobeth expira cada ~4h** y hay que pedirle credenciales nuevas cada vez — esto
  va a repetirse en la exposición de Semana 7; alguien del equipo (idealmente Jobeth) debe tener el
  Learner Lab abierto y listo antes de esa sesión, no durante.
- **Datos reales respaldados** (2026-09-23, post carga de MS1/MS2/MS3): `backups/mysql-20260923-0645.sql`
  (10.9MB), `pg-20260923-0645.sql` (2.9MB), `mongo-20260923-0645.archive` (11.1MB) — si el Lab corta
  sesión y se pierde el volumen de VM-DB, se puede restaurar desde ahí sin volver a correr los seeds.

### Lo que sigue bloqueado y no es mío para resolver solo

- **Amplify** — pendiente que Jobeth lo despliegue desde la consola web (instrucciones ya enviadas);
  si también falla ahí, activar contingencia S3+CloudFront (R1).
- **DA-04** — revisión cruzada del diagrama con Fabricio y Alexander (necesita su input, no solo el mío).
- **EX-08 (informe final)** — `03-backend.tex` (4 pendientes), `05-frontend.tex` (2), `06-data-science.tex`
  (1) siguen con secciones sin cerrar de Guillermo/Mariano/Edinson/Fabricio/Alexander — no es correcto
  que yo las escriba por ellos.
- **EX-10 (subida a Canvas)** — requiere login al LMS del curso, fuera de mi alcance.
- **EX-11/EX-12** — ensayo y exposición presencial, requieren al equipo completo.

### 🟢 Despliegue real de infra (2026-09-22) — cuenta AWS de Jobeth

- **Cambio de cuenta:** el crédito de AWS Academy de Benja se agotó; el equipo despliega desde hoy en
  la cuenta de **Jobeth** (`688609726830`). El Terraform nunca se había aplicado con éxito (no había
  `.tfstate`) — se aplicó por primera vez hoy y se encontraron/corrigieron 3 bugs reales:
  1. `security_groups.tf`: el `name` de un SG no puede empezar con `sg-` (reservado por AWS) — se
     renombró a `aero-sg-*` manteniendo el tag `Name = sg-*` que usan el RUNBOOK y el diagrama.
  2. Mismo archivo: `description` de los SG con em-dash/tildes — la API de EC2 solo acepta ASCII ahí.
  3. `s3.tf`: el provider de AWS intenta leer `GetBucketObjectLockConfiguration` al gestionar
     `aws_s3_bucket`, y una SCP de organización del Learner Lab lo deniega explícitamente — bloqueaba
     el apply completo. Se sacó el bucket de Terraform (`terraform state rm`), se creó/completó por
     CLI (bucket + public-access-block + 3 prefijos), y `s3.tf`/`outputs.tf` quedan documentando esto
     para que no se repita.
- ✅ **BE-INT-01** (VPC+SG), **BE-INT-02** (4 EC2, las 4 `Online` en SSM), **BE-INT-03** (VM-DB: mysql/
  postgres/mongo `Up`/`healthy`, verificado por SSM), **BE-INT-05** (ALB interno creado, targets
  registrados), **BE-INT-06** (API Gateway + VPC Link creados e integrados), **DS-04** (bucket +
  prefijos).
- 🔴 **BE-INT-04 sigue bloqueada:** VM-PROD no tiene ningún contenedor de MS corriendo — no hay
  imágenes en Docker Hub todavía (confirmado por `docker compose ps` vacío en VM-PROD-1). Es el
  bloqueante #1 ahora mismo; sigue con la migración del registro (ver entrada siguiente).
- Outputs: API Gateway `https://0tmopxsjij.execute-api.us-east-1.amazonaws.com` · ALB interno
  `internal-aeropuerto-alb-1804635082.us-east-1.elb.amazonaws.com` · VM-DB `10.0.10.170` · VM-INGESTA
  `10.0.11.207` · VM-PROD `10.0.0.107`/`10.0.1.228` (privadas), con IP pública temporal
  `3.85.35.12`/`34.201.13.75` mientras `sg-vm-prod` sigue abierto (ver nota arriba — cerrar al final).

### 🟢 Registro migrado a Docker Hub + primer despliegue real de los 5 MS (2026-09-22)

- Secrets `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN` configurados en los 5 repos de microservicio; workflow
  `build-push-ghcr.yml` reemplazado por `build-push-dockerhub.yml` (mismo patrón que la plantilla de
  `aeropuerto-infra-deploy`) y pusheado a `main` de cada uno. Las 5 imágenes ya son públicas:
  `btoroled/ms{1..5}-*:latest` (verificado con la API pública de Docker Hub, sin login).
- `docker compose pull && up` corrido por SSM en **VM-PROD-1 y VM-PROD-2**: los 6 contenedores
  (nginx + MS1..MS5) están `Up` en ambas.
- ✅ **BE-INT-04 cerrada.** ✅ **BE-INT-05/06 verificadas de punta a punta:** los 2 targets del ALB
  están `healthy`, y `curl https://0tmopxsjij.execute-api.us-east-1.amazonaws.com/api/pasajeros/health`
  responde **200** desde internet — la cadena completa Internet→API Gateway→VPC Link→ALB→VM-PROD→nginx→MS1
  funciona real, no solo en el papel.
- Chequeo rápido de los otros 4 vía nginx local: MS3 (`/api/infra/health`) → 200 OK. MS2
  (`/api/vuelos/actuator/health` y `/api/vuelos/`) → **500** con el JSON de error propio (el contenedor
  se reporta `healthy` a nivel Docker, así que es un bug de la app de MS2, no de la infra/nginx —
  **avisar a Mariano**). MS4/MS5 devuelven 422/404 en `/health` — probablemente esas rutas no existen
  tal cual en sus apps (Fabricio debería confirmar el path real de cada uno para la evidencia del informe).
- **VM-INGESTA sigue con el compose placeholder** (documentado en el propio archivo) — el pipeline real
  de `aeropuerto-data-science/ingesta/` solo tiene `ingesta-ms1` hecho hoy; `ingesta-ms2`/`ingesta-ms3`
  no existen todavía. Esto bloquea DS-06/07/08 y todo el camino crítico de Athena — **es lo más urgente
  del lado de Data Science ahora mismo**, fuera del alcance de Benja.
