# Arquitectura de Solución

Proyecto Parcial CS2032 — Cloud Computing (2026-2). Entorno: **AWS Academy Learner Lab** (sesión con
temporizador ~4 h, solo `LabRole`, sin IAM propio, algunos servicios restringidos).

Documentos relacionados: [requerimientos](requerimientos.md) · [plan de trabajo](plan-de-trabajo.md) ·
[riesgos](riesgos.md).

---

## 1. Topología de despliegue (1 VPC)

```
                Internet
                   │  HTTPS (execute-api, TLS gestionado por AWS)
          ┌────────▼─────────┐
          │  API Gateway      │  HTTP API, ruta /{proxy+}
          │  (público)        │
          └────────┬─────────┘
                   │  VPC Link
          ┌────────▼─────────┐   subred privada
          │  ALB interno      │   (o NLB si VPC Link lo exige con LabRole)
          └───┬──────────┬───┘
              │          │
      ┌───────▼──┐   ┌───▼───────┐   subred privada — 2 VMs de PRODUCCIÓN (t3.small)
      │ VM-PROD-1│   │ VM-PROD-2 │   cada una: docker compose con
      │ nginx:80 │   │ nginx:80  │     nginx (reverse proxy por path)
      │ MS1..MS5 │   │ MS1..MS5  │     + MS1 + MS2 + MS3 + MS4 + MS5
      └────┬─────┘   └─────┬─────┘
           └──────┬────────┘
                  │  3306 / 5432 / 27017
          ┌───────▼────────┐   subred privada — VM de BASES DE DATOS (t3.medium)
          │  VM-DB          │   docker compose: mysql:8 + postgres:16 + mongo:7 + volúmenes
          └────────────────┘

  Subred privada aparte — VM-INGESTA (t3.small): docker compose con 3 contenedores
  Python de ingesta → escribe CSV/JSON a  s3://<bucket>/raw/ms{1,2,3}/<tabla>/<fecha>/
  Glue (crawlers) → catálogo  aeropuerto_lake  → Athena (consultas + vistas)  → MS5

  Acceso a instancias: AWS SSM Session Manager (sin puerto 22 público). Sin bastión.
```

### Reglas de seguridad (Security Groups)

| SG | Entrada permitida | Notas |
|---|---|---|
| `sg-alb` | Desde `sg-apigw-vpclink`, puerto 80 | ALB **interno**, sin IP pública |
| `sg-vm-prod` | Desde `sg-alb`, puerto 80 | Las 2 VMs de producción |
| `sg-vm-db` | Desde `sg-vm-prod`, puertos 3306 / 5432 / 27017 | **Sin IP pública** |
| `sg-vm-ingesta` | — (solo salida) | Salida a `sg-vm-db` (lectura) y a S3 (VPC endpoint / NAT) |

Todas las instancias usan el perfil `LabInstanceProfile` / rol `LabRole`.

**Verificación de "privado":** desde fuera de la VPC, `nc -zv <alb-dns> 80` y `mysql -h <db-vm>` deben
**fallar**; solo la URL de API Gateway responde. Ver [verificación](verificacion.md).

---

## 2. nginx como gateway local (por VM-PROD)

Un contenedor nginx enruta por *path* a los 5 servicios locales, de modo que el ALB tiene un único
target (`:80`) y no hace falta un *target group* por servicio.

| Path | Servicio | Puerto interno |
|---|---|---|
| `/api/pasajeros/*` | MS1 | 8001 |
| `/api/vuelos/*` | MS2 | 8002 |
| `/api/infra/*` | MS3 | 8003 |
| `/api/manifiesto/*` | MS4 | 8004 |
| `/api/analitica/*` | MS5 | 8005 |
| `/docs/*` | página Swagger-UI agregada | 8080 |

El `docker compose` de cada VM-PROD levanta: `nginx` + `ms1` + `ms2` + `ms3` + `ms4` + `ms5` +
`swagger-aggregator`. Las imágenes se descargan de GHCR por tag.

---

## 3. Servicios AWS usados (para el diagrama `draw.io`)

| Categoría | Servicios |
|---|---|
| Cómputo | EC2 × 4 (2 VM-PROD, 1 VM-DB, 1 VM-INGESTA) |
| Red | VPC, subredes públicas/privadas, Internet Gateway, NAT Gateway, Security Groups, **API Gateway (HTTP API)** + **VPC Link**, **ALB** (o NLB) interno |
| Almacenamiento | **S3** (data lake + backups de volúmenes de BD) |
| Analítica | **AWS Glue** (Data Catalog + Crawlers), **Amazon Athena** |
| Frontend | **AWS Amplify** (contingencia: S3 static website + CloudFront) |
| Operación | **SSM Session Manager**, **CloudWatch Logs** |
| Imágenes | **GHCR** (GitHub Container Registry) — preferido sobre ECR por los límites de IAM del Learner Lab |

El diagrama vive en `diagramas/arquitectura-solucion.drawio` y se exporta a PNG para el informe.

---

## 4. Flujo de datos

1. **Operación (escritura):** Frontend → API Gateway (HTTPS) → VPC Link → ALB interno → nginx → MS1..MS5.
   MS1/MS3 llaman a MS2 para validar vuelos. MS4 agrega llamando a MS1+MS2+MS3.
2. **Persistencia:** MS1→MySQL, MS2→PostgreSQL, MS3→MongoDB (los 3 motores en la VM-DB privada).
3. **Ingesta (batch, una vez):** VM-INGESTA corre los 3 contenedores → *pull* 100% de cada BD →
   CSV/JSON a `s3://<bucket>/raw/msX/`.
4. **Catálogo:** crawlers de Glue sobre `raw/msX/` → tablas en la base `aeropuerto_lake`.
5. **Analítica:** MS5 ejecuta consultas Athena (join entre tablas de ms1/ms2/ms3) y lee vistas;
   el Frontend muestra el dashboard consumiendo MS5.

---

## 5. Reproducibilidad ante el temporizador de sesión

- Toda la infraestructura se define en `aeropuerto-infra-deploy` (Terraform o scripts `aws-cli` + `user-data`).
- Las VMs no guardan estado en disco local salvo los volúmenes de BD, que se respaldan a S3 con
  `mysqldump` / `pg_dump` / `mongodump` al cerrar la sesión.
- Runbook de reinicio (`aeropuerto-infra-deploy/RUNBOOK.md`): objetivo **< 15 min** para dejar todo
  operativo tras un corte (`terraform apply` o script → `docker compose up -d` en cada VM → restaurar
  dumps si aplica).
