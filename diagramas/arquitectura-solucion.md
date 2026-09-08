# Arquitectura de solución — boceto (DA-01)

Boceto v0 de la topología, para alinear al equipo en F0. Se mantiene actualizado hasta que en **F2**
se produzca el `arquitectura-solucion.drawio` + PNG final para el informe (DA-03).

Fuente de verdad: [`docs/arquitectura.md`](../docs/arquitectura.md) · decisiones pendientes en
[`docs/aws-learner-lab-hallazgos.md`](../docs/aws-learner-lab-hallazgos.md).

```mermaid
flowchart TB
    User([Usuario / navegador])
    Amplify["AWS Amplify<br/>(SPA React)<br/><i>o S3 + CloudFront</i>"]
    GHCR[("GHCR<br/>imágenes Docker")]

    User --> Amplify
    Amplify -->|HTTPS| APIGW

    subgraph AWS["AWS · us-east-1 · VPC 10.0.0.0/16"]
        APIGW["API Gateway<br/>HTTP API · /{proxy+}"]

        subgraph PUB["Subred pública"]
            NAT["NAT Gateway"]
            IGW["Internet Gateway"]
        end

        APIGW -->|VPC Link| ALB

        subgraph PRIV["Subredes privadas (2 AZ)"]
            ALB["ALB interno :80"]

            subgraph PROD["VM-PROD-1 y VM-PROD-2 · t3.small"]
                NGINX["nginx (reverse proxy por path)"]
                MS1["MS1 pasajeros :8001<br/>Python/FastAPI"]
                MS2["MS2 vuelos :8002<br/>Java/Spring"]
                MS3["MS3 infra :8003<br/>Node/Express"]
                MS4["MS4 manifiesto :8004<br/>Python/FastAPI"]
                MS5["MS5 analítica :8005<br/>Python/boto3"]
                NGINX --> MS1 & MS2 & MS3 & MS4 & MS5
            end

            subgraph DB["VM-DB · t3.medium"]
                MYSQL[("MySQL 8")]
                PG[("PostgreSQL 16")]
                MONGO[("MongoDB 7")]
            end

            subgraph ING["VM-INGESTA · t3.small"]
                I1["ingesta-ms1"]
                I2["ingesta-ms2"]
                I3["ingesta-ms3"]
            end

            ALB --> NGINX
            MS1 --> MYSQL
            MS2 --> PG
            MS3 --> MONGO
            MS1 -.->|valida vuelo| MS2
            MS3 -.->|valida vuelo| MS2
            MS4 -.->|agrega| MS1 & MS2 & MS3
            I1 --> MYSQL
            I2 --> PG
            I3 --> MONGO
        end

        S3[("S3<br/>mla-aeropuerto-lake<br/>raw/ms1|ms2|ms3")]
        GLUE["AWS Glue<br/>Data Catalog + Crawlers"]
        ATHENA["Amazon Athena<br/>5 consultas + 2 vistas"]

        I1 & I2 & I3 -->|CSV/JSON| S3
        S3 --> GLUE --> ATHENA
        ATHENA --> MS5

        NAT --> IGW
    end

    PROD -.->|pull imágenes| GHCR
    ING -.->|pull imágenes| GHCR

    SSM["SSM Session Manager"] -.->|shell sin puerto 22| PROD & DB & ING
    CW["CloudWatch Logs"] -.-> PROD & DB & ING
```

## Capas (para el diagrama final)

| Capa | Elementos |
|---|---|
| Red | VPC, 2 subredes públicas, 2 privadas, IGW, NAT, 5 Security Groups, API Gateway + VPC Link, ALB interno |
| Cómputo Backend | EC2 ×2 VM-PROD (nginx + MS1..MS5) |
| Datos Backend | EC2 VM-DB (MySQL + PostgreSQL + MongoDB) |
| Data Science | EC2 VM-INGESTA → S3 → Glue → Athena → MS5 |
| Frontend | Amplify (o S3 + CloudFront) → API Gateway |
| Operación | SSM Session Manager, CloudWatch Logs |
| Externo | GHCR (registro de imágenes) |

## Flujos

1. `Usuario → Amplify → API Gateway (HTTPS) → VPC Link → ALB → nginx → MSx`
2. `MS1→MySQL`, `MS2→PostgreSQL`, `MS3→MongoDB`; `MS1/MS3→MS2` (valida vuelo); `MS4→MS1+MS2+MS3` (agrega)
3. `VM-INGESTA → S3 (raw/msX/)` (pull 100%, una vez)
4. `S3 → Glue crawlers → aeropuerto_lake → Athena → MS5`

## Pendiente para DA-02 / DA-03

- Confirmar **ALB vs NLB** y **VPC Link sí/no** con los hallazgos del Learner Lab.
- Reemplazar por `arquitectura-solucion.drawio` con **nombres/IDs reales** de recursos y exportar **PNG**
  para `informe/` y `ppt/` (F2).
