# Hallazgos del AWS Academy Learner Lab

Se completa en la **Fase 0** (antes del Sáb 6-Set). Dueño: Benja.

Objetivo: saber qué servicios están disponibles con `LabRole` y con qué límites, antes de comprometer
la arquitectura.

- **Región:** `us-east-1` · **Fecha de validación:** _(pendiente)_ · **Crédito al inicio:** _$____
- Procedimiento detallado de creación en
  [`aeropuerto-infra-deploy/RUNBOOK.md`](https://github.com/Cloud-MLA/aeropuerto-infra-deploy/blob/main/RUNBOOK.md).

Leyenda de estado: ✅ funciona · ⚠️ funciona con límite/rodeo · ❌ no disponible · ⬜ sin probar.

---

## Checklist de validación (en este orden)

### 1 · Red y cómputo (EC2 / VPC)

| # | Qué probar | Cómo (consola) | Estado | Resultado / notas |
|---|---|---|---|---|
| 1.1 | Crear VPC + subredes + IGW + NAT | VPC → Create VPC → "VPC and more" (2 AZ, 2 pub, 2 priv, NAT en 1 AZ, endpoint S3) | ⬜ | |
| 1.2 | Endpoint S3 Gateway | incluido en el asistente anterior | ⬜ | |
| 1.3 | Crear 5 Security Groups con referencias entre sí | VPC → Security Groups | ⬜ | |
| 1.4 | Lanzar EC2 `t3.small` con `LabInstanceProfile` | EC2 → Launch instance (AL2023, sin key pair, IP pública off) | ⬜ | ¿deja `t3.small`? ¿`t3.medium`? |
| 1.5 | `user-data` instala Docker | ver script en RUNBOOK 1.4 | ⬜ | `docker --version` dentro de la instancia |
| 1.6 | **SSM Session Manager** (sin puerto 22) | EC2 → instancia → Connect → Session Manager | ⬜ | ¿abre shell en < 3 min? |

### 2 · Almacenamiento (S3)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 2.1 | Crear bucket `mla-aeropuerto-lake` | S3 → Create bucket (block public access ON) | ⬜ | nombre final del bucket: |
| 2.2 | Crear prefijos `raw/`, `athena-results/`, `backups/` | S3 → Create folder | ⬜ | |
| 2.3 | Subir/leer un objeto de prueba | S3 → Upload / `aws s3 cp` desde una EC2 | ⬜ | ¿la EC2 escribe con el rol, sin credenciales? |

### 3 · Analítica (Glue / Athena)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 3.1 | Crear base de datos Glue `aeropuerto_lake` | Glue → Databases → Add database | ⬜ | |
| 3.2 | Crear un crawler con `LabRole` | Glue → Crawlers → Create (no ejecutar aún) | ⬜ | ¿deja elegir `LabRole`? |
| 3.3 | Athena: fijar output y correr `SELECT 1;` | Athena → Settings → result location + Query editor | ⬜ | |
| 3.4 | Athena: crear una vista de prueba y borrarla | `CREATE VIEW v_probe AS SELECT 1; DROP VIEW v_probe;` | ⬜ | |

### 4 · Borde (API Gateway / VPC Link)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 4.1 | Crear un HTTP API de prueba y borrarlo | API Gateway → Create API → HTTP API | ⬜ | |
| 4.2 | **VPC Link para HTTP API** | API Gateway → VPC links → Create (subredes privadas, SG `sg-apigw-vpclink`) | ⬜ | ¿queda `AVAILABLE`? ¿exige NLB? ¿`LabRole` alcanza? |
| 4.3 | Integración privada API Gateway → ALB | (requiere ALB; se prueba en F1) | ⬜ | |

### 5 · Frontend (Amplify / CloudFront)

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 5.1 | Amplify: conectar GitHub y ver repos | Amplify → Deploy an app → GitHub | ⬜ | ¿autoriza la org `Cloud-MLA`? |
| 5.2 | Amplify: crear app (sin desplegar aún) | Amplify → New app | ⬜ | |
| 5.3 | Contingencia: S3 static website + CloudFront | S3 → Properties → Static website hosting / CloudFront → Create distribution | ⬜ | solo si 5.1/5.2 fallan |

### 6 · Operación

| # | Qué probar | Cómo | Estado | Resultado / notas |
|---|---|---|---|---|
| 6.1 | CloudWatch Logs desde una EC2 | agente CloudWatch o `docker` log driver | ⬜ | |
| 6.2 | Presupuesto restante | banner del Learner Lab | ⬜ | $ al terminar F0: |
| 6.3 | Duración real de la sesión antes del corte | observación | ⬜ | ~___ h |

---

## Decisiones tomadas a partir de los hallazgos

_(completar tras la validación)_

| Decisión | Opción elegida | Porque… |
|---|---|---|
| Balanceador interno | ALB / NLB | |
| Exposición HTTPS | API Gateway + VPC Link / ALB público solo demo (R2) | |
| Frontend | Amplify / S3 + CloudFront (R1) | |
| Egress de las VMs privadas | NAT Gateway / VPC endpoints / IP pública temporal | |
| Formato de datos para Glue | CSV / Parquet (R9) | |

---

## Incidencias encontradas

_(lista libre: permisos que faltan, servicios bloqueados, cuotas, etc.)_

-
