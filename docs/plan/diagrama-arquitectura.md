# Plan de trabajo — Diagrama de Arquitectura de Solución

**Rúbrica:** 1 pt · **Entrega:** Hito 2 (dentro del informe).
Documentos base: [arquitectura](../arquitectura.md) · [plan general](../plan-de-trabajo.md).

---

## 1. Entregables y Definición de Terminado

- [ ] `diagramas/arquitectura-solucion.drawio` (editable) + **export PNG** incluido en el informe y el PPT.
- [ ] Incluye **todos** los servicios AWS efectivamente usados en **Backend + Frontend + Data Science**.
- [ ] Muestra las 3 capas del enunciado con sus flujos: operación (request), persistencia, ingesta/analítica.
- [ ] Agrupación visual por subred pública/privada + leyenda + notas de Learner Lab.
- [ ] Coincide con lo realmente desplegado (nombres de recursos reales en la versión final).

**DoD:** el diagrama se puede leer sin explicación oral; un evaluador identifica cada servicio, su capa
y el sentido de cada flecha; no hay servicios "fantasma" ni omitidos respecto al despliegue real.

---

## 2. Asignación

| Trabajo | Responsable | Consultado |
|---|---|---|
| Diagrama completo (draw.io) | **Lead** | Dev D (Data Science), Dev E (Frontend) |
| Revisión final con el equipo | Todos | — |

---

## 3. Contenido mínimo del diagrama

| Capa | Elementos |
|---|---|
| Red | VPC, subred pública, subredes privadas, Internet Gateway, NAT Gateway, Security Groups (etiquetados) |
| Borde | **API Gateway (HTTP API)** + **VPC Link**; **ALB/NLB interno** |
| Cómputo Backend | EC2 **VM-PROD-1** y **VM-PROD-2** (nginx + MS1..MS5 en `docker compose`) |
| Datos Backend | EC2 **VM-DB** (MySQL + PostgreSQL + MongoDB en contenedores, subred privada) |
| Data Science | EC2 **VM-INGESTA** (3 contenedores) → **S3** → **AWS Glue** (Data Catalog + Crawlers) → **Amazon Athena** → MS5 |
| Frontend | **AWS Amplify** (o **S3 static website + CloudFront** si contingencia) → API Gateway |
| Operación | **SSM Session Manager**, **CloudWatch Logs** |
| Externo | **GHCR** (registro de imágenes) — fuera de AWS, con nota |
| Flujos (flechas) | 1) Internet→APIGW→VPCLink→ALB→nginx→MS  2) MS→VM-DB  3) MS1→MS2, MS3→MS2, MS4→MS1/2/3  4) VM-INGESTA→S3  5) Glue→Athena→MS5  6) Frontend→APIGW |

---

## 4. Tareas

| ID | Tarea | Resp. | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|---|
| DA-01 | Boceto v0 con la topología planeada (guía para el equipo) | Lead | 0.5d | F0 | [arquitectura](../arquitectura.md) | PNG compartido en el repo docs |
| DA-02 | v1 tras validar Learner Lab: fijar ALB vs NLB, Amplify vs CloudFront, VPC Link sí/no | Lead | 0.5d | F1 | [hallazgos Learner Lab](../aws-learner-lab-hallazgos.md) | Diagrama refleja las decisiones tomadas |
| DA-03 | v2 final: nombres reales de recursos, todos los servicios usados, IDs de subred/SG | Lead | 0.5d | F2 | despliegue F2 | Coincide 1:1 con lo desplegado |
| DA-04 | Revisión cruzada con Dev D (DS) y Dev E (Frontend) | Lead + Dev D + Dev E | 0.25d | F2 | DA-03 | Sin observaciones pendientes |
| DA-05 | Export PNG + insertar en informe y PPT | Lead | 0.25d | F2/F3 | DA-04 | PNG en `informe/` y `ppt/` |

**Carga total ≈ 2 d (Lead).**

---

## 5. Mapa a hitos

- **Hito 1:** basta el boceto v0/v1 como material de la exposición virtual (no puntúa aún, pero ayuda a
  explicar el avance).
- **Hito 2:** v2 final dentro del informe — es donde se otorga el punto.

---

## 6. Dependencias y riesgos

| Depende de | Riesgo | Mitigación |
|---|---|---|
| Hallazgos del Learner Lab (F0) | Diagrama se hace sobre una arquitectura que luego no se puede desplegar | No congelar hasta DA-02 (post-validación) |
| Despliegue real F2 | Diagrama desactualizado respecto a lo desplegado | DA-03 se hace **después** de BE-INT-08; el Lead actualiza si algo cambió |
| Decisión Amplify/CloudFront y ALB/NLB | Mostrar un servicio que no se usó | Reflejar solo lo efectivamente desplegado; anotar contingencias como alternativa |
