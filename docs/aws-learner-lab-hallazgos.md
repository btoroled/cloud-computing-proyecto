# Hallazgos del AWS Academy Learner Lab

Se completa en la **Fase 0** (antes del Sáb 6-Set). Dueño: Benja.

Objetivo: saber qué servicios están disponibles con `LabRole` y con qué límites, antes de comprometer
la arquitectura.

- **Región:** `us-east-1` · **Fecha de validación:** `2026-09-12` · **Crédito:** ~$4 usados de $50 iniciales (~$46 restantes) al día de hoy.
- Procedimiento detallado de creación en
  [`aeropuerto-infra-deploy/RUNBOOK.md`](https://github.com/Cloud-MLA/aeropuerto-infra-deploy/blob/main/RUNBOOK.md) —
  este checklist se llenó cruzando ese runbook (Parte 1, pasos 1.1–1.11) con lo reportado por Benja.

Leyenda de estado: ✅ funciona · ⚠️ funciona con límite/rodeo · ❌ no disponible · ⬜ sin probar.

---

## Checklist de validación (en este orden)

### 1 · Red y cómputo (EC2 / VPC)

| # | Qué probar | Cómo (consola) | Estado | Resultado / notas |
|---|---|---|---|---|
| 1.1 | Crear VPC + subredes + IGW + NAT | VPC → Create VPC → "VPC and more" (2 AZ, 2 pub, 2 priv, NAT en 1 AZ, endpoint S3) | ✅ | Creada tal como estaba planeada (VPC `aeropuerto` 10.0.0.0/16, 2 públicas + 2 privadas, 1 NAT). Sin límites encontrados. |
| 1.2 | Endpoint S3 Gateway | incluido en el asistente anterior | ✅ | Viene en el mismo asistente de 1.1 (`aws_vpc_endpoint "s3"` en terraform); usado por VM-INGESTA/VM-DB para llegar a S3 sin salir por NAT. |
| 1.3 | Crear 5 Security Groups con referencias entre sí | VPC → Security Groups | ✅ | Los 5 (`sg-apigw-vpclink`, `sg-alb`, `sg-vm-prod`, `sg-vm-db`, `sg-vm-ingesta`) creados y referenciados entre sí por ID, sin CIDRs a mano. |
| 1.4 | Lanzar EC2 `t3.small` con `LabInstanceProfile` | EC2 → Launch instance (sin key pair, IP pública off) | ✅ | **AMI real: Cloud9 Ubuntu 22 (`ami-01112e374e42e3f3c`)**, no Amazon Linux 2023 como estaba planeado — es la AMI usada en clases. El Lab dejó `t3.small` (VM-PROD-1/2, VM-INGESTA) y también `t3.medium` (VM-DB). |
| 1.5 | `user-data` instala Docker | ver script en RUNBOOK 1.4 | ✅ | Con `apt`/`ubuntu` (no `dnf`/`ec2-user`, por ser Ubuntu). Además crea usuario `ssm-user` — ver 1.6. |
| 1.6 | **SSM Session Manager** (sin puerto 22) | EC2 → instancia → Connect → Session Manager | ✅ | Abre shell en las 4 instancias; a veces tarda 2–3 min en aparecer conectable. Conecta como `ssm-user`, no `ubuntu`/`ec2-user`. La sesión arranca en `sh`, no `bash`. |

### 2 · Almacenamiento (S3)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 2.1 | Crear bucket `mla-aeropuerto-lake` | S3 → Create bucket (block public access ON) | ✅ | nombre final del bucket: **`mla-aeropuerto-lake`** — no chocó con otro equipo, no hizo falta prefijo propio. |
| 2.2 | Crear prefijos `raw/`, `athena-results/`, `backups/` | S3 → Create folder | ✅ | |
| 2.3 | Subir/leer un objeto de prueba | S3 → Upload / `aws s3 cp` desde una EC2 | ✅ | Confirmado en la práctica: VM-PROD y VM-DB hacen `aws s3 cp` sin configurar credenciales — `LabInstanceProfile` les da el permiso automáticamente (usado para pasar `docker-compose.yml`/`nginx.conf` y para los backups de BD). |

### 3 · Analítica (Glue / Athena)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 3.1 | Crear base de datos Glue `aeropuerto_lake` | Glue → Databases → Add database | ⬜ | Explícitamente pospuesto a F2 en el RUNBOOK (1.10) — se corre cuando la ingesta ya haya subido datos a S3. |
| 3.2 | Crear un crawler con `LabRole` | Glue → Crawlers → Create (no ejecutar aún) | ⬜ | Ídem — F2. |
| 3.3 | Athena: fijar output y correr `SELECT 1;` | Athena → Settings → result location + Query editor | ⬜ | Ídem — F2. |
| 3.4 | Athena: crear una vista de prueba y borrarla | `CREATE VIEW v_probe AS SELECT 1; DROP VIEW v_probe;` | ⬜ | Ídem — F2. |

### 4 · Borde (API Gateway / VPC Link)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 4.1 | Crear un HTTP API de prueba y borrarlo | API Gateway → Create API → HTTP API | ✅ | Se creó directo la API real (`aeropuerto-api`), sin una de prueba aparte. |
| 4.2 | **VPC Link para HTTP API** | API Gateway → VPC links → Create (subredes privadas, SG `sg-apigw-vpclink`) | ✅ | Queda `AVAILABLE` (~2 min). No exigió NLB — funciona contra el ALB. `LabRole`/`LabInstanceProfile` alcanzaron, sin bloqueos de IAM. |
| 4.3 | Integración privada API Gateway → ALB | validado hoy con las 4 EC2 y el ALB ya arriba | ⚠️ | Funciona (`curl` a la Invoke URL responde), pero con un rodeo: el asistente a veces crea la ruta `ANY /{proxy+}` **sin** asociarle la integración — hay que abrir la ruta y verificar/asociarla a mano antes de probar. |

### 5 · Frontend (Amplify / CloudFront)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 5.1 | Amplify: conectar GitHub y ver repos | Amplify → Deploy an app → GitHub | ⬜ | **Decisión ya tomada: Amplify** (lo pide el enunciado) — falta correr este paso para confirmar que autoriza la org `Cloud-MLA`. |
| 5.2 | Amplify: crear app (sin desplegar aún) | Amplify → New app | ⬜ | |
| 5.3 | Contingencia: S3 static website + CloudFront | S3 → Properties → Static website hosting / CloudFront → Create distribution | — | Descartada mientras Amplify funcione — no hace falta probarla salvo que 5.1/5.2 fallen. |

### 6 · Operación

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 6.1 | CloudWatch Logs desde una EC2 | agente CloudWatch o `docker` log driver | ⬜ | Todavía no configurado (es BE-INT-09, queda para F2). |
| 6.2 | Presupuesto restante | banner del Learner Lab | ✅ | ~$46 de $50 restantes (~$4 usados) al 2026-09-12. |
| 6.3 | Duración real de la sesión antes del corte | observación | ✅ | ~4 horas — coincide con el timer estándar del Learner Lab (RUNBOOK §0: "temporizador ~4 h apaga las instancias"). |

---

## Decisiones tomadas a partir de los hallazgos

| Decisión | Opción elegida | Porque… |
|---|---|---|
| Balanceador interno | **ALB** | Enruta bien con VPC Link (`HTTP_PROXY`); no hizo falta NLB. |
| Exposición HTTPS | **API Gateway + VPC Link** | Validado extremo a extremo hoy (API Gateway → VPC Link → ALB → nginx → MSx). |
| Frontend | **Amplify** | Lo pide el enunciado — se descarta S3+CloudFront (R1) salvo que Amplify falle al probarlo (5.1/5.2). |
| Egress de las VMs privadas | **NAT Gateway** (+ endpoint S3 Gateway para tráfico a S3) | Es lo que crea el asistente "VPC and more"; VM-INGESTA llega a S3 por el endpoint sin pasar por NAT. |
| Formato de datos para Glue | ⬜ pendiente (F2, junto con 3.1–3.4) | Se decide cuando arranque la ingesta real hacia S3. |

---

## Incidencias encontradas

- **AMI planeada no disponible tal cual**: se usó Cloud9 Ubuntu 22 (`ami-01112e374e42e3f3c`, la de clases) en vez de Amazon Linux 2023 — cambia `apt`/`ubuntu` por `dnf`/`ec2-user` en el `user-data`.
- **Puertos nativos de esa AMI chocan con Docker**: trae `apache2` (80) y `mysql` (3306) corriendo — hay que `stop`/`disable` antes de `docker compose up` en VM-PROD y VM-DB, o falla `address already in use`.
- **SSM conecta como `ssm-user`**, no `ubuntu`/`ec2-user` — el `user-data` lo crea y lo mete al grupo `docker`. Requiere `LabInstanceProfile` en la instancia o no aparece conectable.
- **La sesión de SSM arranca en `sh`, no `bash`** — patrones como `read -s -p "..." VAR` fallan con `Illegal option -s`; hay que correr `bash` primero.
- **Contraseñas de BD limitadas**: `$`, `\`, comillas o backticks rompen la interpolación de `.env` de Compose o el entrypoint de `mysql:8` — usar solo letras/números/`-`/`_`.
- **Pegar archivos largos directo en la consola de SSM puede corromper el contenido** sin error visible — mejor subir una vez a S3 desde la máquina local y bajar con `aws s3 cp` dentro de la instancia.
- **GHCR necesita un PAT clásico (`ghp_...`)**, no fine-grained — un fine-grained sin la org como "Resource owner" hace login OK pero el `pull` falla con 404 aunque la imagen exista.
- **Healthcheck no es uniforme entre microservicios**: `ms1`/`ms3`/`ms4`/`ms5` usan `/api/<ruta>/health`; MS2 (Spring) usa `/api/vuelos/actuator/health`.
- **La ruta `ANY /{proxy+}` de API Gateway a veces queda sin integración asociada** — verificarlo a mano en Routes antes de dar por fallido un `curl`.
- **El asistente "VPC and more" crea una route table privada por AZ**, no una compartida — si se recrea el NAT Gateway, hay que actualizar la ruta `0.0.0.0/0` en ambas.
- **El nombre de bucket S3 es global** — en teoría `mla-aeropuerto-lake` podía chocar con el de otro equipo; en la práctica no chocó, se quedó con ese nombre tal cual.
- **`swagger-aggregator` sin imagen todavía** (bloqueado por Frontend) — queda comentado en `nginx.conf` hasta que exista, si no nginx falla al arrancar por no poder resolver ese upstream.
