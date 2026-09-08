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
| ☐ | — | Validar Learner Lab en orden EC2+VPC → S3 → Glue+Athena → API Gateway+VPC Link → Amplify y escribir [`aws-learner-lab-hallazgos.md`](../../aws-learner-lab-hallazgos.md) | 0.5d | Learner Lab | Doc de hallazgos con qué hay y qué no |
| 🟡 | BE-TX-01 | Acordar y versionar `contratos/enums.md` y rangos de ID (con todo el equipo) | 0.5d | — | Archivo aprobado en repo docs |
| 🟡 | BE-TX-02 | Plantilla de repo de microservicio (estructura, `.editorconfig`, `.env.example`, `README` base) | 0.5d | — | 5 repos creados desde plantilla |
| 🟡 | BE-TX-04 | Imagen base Docker por lenguaje (Py 3.12-slim, Temurin 21, Node 20-alpine) + healthcheck | 0.5d | — | Imágenes construyen y publican a GHCR |
| ✅ | BE-TX-08 | Contrato de errores común (JSON de error, códigos 400/404/422/502) | 0.25d | — | Documentado en `contratos/` |
| ☐ | BE-INT-01 | VPC + subredes privadas + IGW/NAT + Security Groups | 1d | Learner Lab OK | SGs mínimos aplicados |
| ☐ | BE-INT-02 | 3 EC2: VM-PROD-1, VM-PROD-2, VM-DB + acceso por SSM (sin puerto 22 público) | 0.5d | BE-INT-01 | Acceso por SSM funcionando |
| ☐ | DA-01 | Boceto v0 del diagrama con la topología planeada | 0.5d | [arquitectura](../../arquitectura.md) | PNG compartido en el repo docs |

## F1 — Núcleo + deploy v1 (Sáb 6 – Sáb 12 · Hito 1)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-TX-05 | GitHub Actions: build + push a GHCR en tag `vX.Y` | 0.5d | BE-TX-02 | Tag dispara imagen `ghcr.io/btoroled/<repo>:<tag>` |
| ☐ | BE-TX-06 | `nginx.conf` reverse proxy por path (`/api/pasajeros`, `/api/vuelos`, …) | 0.5d | contratos | nginx enruta a los 5 servicios locales |
| ☐ | BE-INT-03 | VM-DB: `compose` con `mysql:8` + `postgres:16` + `mongo:7` + volúmenes | 0.5d | BE-INT-02 | Los 3 motores `Up`; solo VM-PROD conecta |
| ☐ | BE-INT-04 | VM-PROD ×2: `compose` con nginx + MS1..MS5 (pull de GHCR) | 1d | BE-TX-06, imágenes | 5 servicios `Up` en ambas VMs |
| ☐ | BE-INT-05 | ALB interno (o NLB) → target de las 2 VM-PROD:80 | 0.5d | BE-INT-04 | Balancea entre las 2 VMs |
| ☐ | BE-INT-06 | API Gateway HTTP API + VPC Link → ALB; ruta `/{proxy+}` | 1d | BE-INT-05, Learner Lab | URL pública HTTPS responde `/api/*/health` |
| ☐ | BE-INT-07 | `RUNBOOK.md` de reinicio tras corte (< 15 min) + script de dumps a S3 | 0.5d | BE-INT-03 | Reinicio cronometrado documentado |
| ☐ | DS-04 | Bucket S3 + estructura de prefijos `raw/ms{1,2,3}/` + política `LabRole` | 0.25d | Learner Lab | `aws s3 ls` muestra el bucket |
| ☐ | DS-05 | VM-INGESTA (EC2 `t3.small`) + `compose` de los 3 contenedores + acceso lectura a VM-DB | 0.5d | BE-INT-02 | `compose` levanta; alcanza la VM-DB |
| ☐ | DA-02 | Diagrama v1 tras validar Learner Lab (ALB vs NLB, Amplify vs CloudFront, VPC Link sí/no) | 0.5d | hallazgos Learner Lab | Refleja las decisiones tomadas |
| ☐ | EX-01 | Plantilla del informe (Word/Docs) con las 9 secciones + placeholders de evidencia | 0.5d | — | Plantilla compartida |
| ☐ | EX-02 | Slides de avance Hito 1 (1–2 por integrante) + consolidación | 1d | avance F1 | PPT de avance listo |
| ☐ | EX-03 | Guion de la demo corta de Hito 1 + ensayo interno | 0.5d | deploy v1 | Demo de ≤10 min ensayada |
| ☐ | EX-04 | **Exposición virtual con ACL** (con todo el equipo) | 0.5d | EX-02/03 | Realizada; feedback del ACL anotado |

## F2 — Completar, 20k, endurecer (Sáb 13 – Vie 19)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | BE-INT-08 | Prueba de humo E2E por la URL pública + verificación de "privado" (ALB/BD inaccesibles) | 0.5d | todo F2 | Script E2E verde |
| ☐ | BE-INT-09 | Logs de contenedores a CloudWatch (o local) + healthchecks en `compose` | 0.5d | BE-INT-04 | Logs visibles; healthchecks activos |
| ☐ | DA-03 | Diagrama v2 final: nombres reales de recursos, todos los servicios, IDs de subred/SG | 0.5d | despliegue F2 | Coincide 1:1 con lo desplegado |
| ☐ | DA-04 | Revisión cruzada del diagrama con Fabricio (DS) y Alexander (Frontend) | 0.25d | DA-03 | Sin observaciones pendientes |
| ☐ | EX-07 | PPT resumen del Hito 2 (estructura + 2–3 slides por integrante) | 1d | EX-06 | PPT completo |
| ☐ | EX-11 | Ensayo general de la demo en vivo (Vie 19) + reparto de quién muestra qué (con todos) | 0.5d | code freeze | Demo de ~15 min cronometrada |

## F3 — Entrega y exposición (Sáb 20 / Semana 7)

| ✔ | ID | Tarea | Est. | Depende de | DoD |
|---|---|---|---|---|---|
| ☐ | DA-05 | Export PNG del diagrama + insertar en informe y PPT | 0.25d | DA-04 | PNG en `informe/` y `ppt/` |
| ☐ | EX-08 | Consolidación final del informe + revisión cruzada + export **PDF** | 1d | EX-06 | PDF final revisado por ≥2 personas |
| ☐ | EX-09 | Verificar `INDEX.md` (todos los repos públicos y accesibles) | 0.25d | repos | Enlaces abren sin login |
| ☐ | EX-10 | **Subida a Canvas** (informe PDF + PPT + enlaces) antes del Dom 20-Set 23:59 | 0.25d | EX-07/08/09 | Entrega confirmada |
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

### 🟡 BE-TX-02 / BE-TX-04 / BE-TX-05 — plantilla + imágenes base + CI

- En progreso: `aeropuerto-infra-deploy/plantilla/` (`.editorconfig`, `.env.example`, README base,
  Dockerfiles Py/Java/Node con healthcheck, workflow `build-push-ghcr.yml`).
