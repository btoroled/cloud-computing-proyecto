# Índice de repositorios — Proyecto Parcial CS2032 (2026-2)

Todos los repositorios son **públicos** en GitHub. Este archivo se completa a medida que se crean
(Fase 0 del plan).

| # | Repo | Componente | Stack | Dueño | URL |
|---|---|---|---|---|---|
| 0 | `cloud-computing-proyecto` | Documentación | Markdown / draw.io | Lead | https://github.com/btoroled/cloud-computing-proyecto |
| 1 | `ms1-pasajeros-api` | Backend — MS1 Pasajeros / Tickets | Python + FastAPI + MySQL 8 | Dev A | _pendiente_ |
| 2 | `ms2-vuelos-api` | Backend — MS2 Vuelos / Operaciones | Java + Spring Boot + PostgreSQL 16 | Dev B | _pendiente_ |
| 3 | `ms3-infraestructura-api` | Backend — MS3 Infraestructura / Incidencias | Node.js + Express + MongoDB 7 | Dev C | _pendiente_ |
| 4 | `ms4-manifiesto-api` | Backend — MS4 Manifiesto de Vuelo (sin BD) | Python + FastAPI | Dev D | _pendiente_ |
| 5 | `ms5-analitica-api` | Backend — MS5 Analítico (Athena) | Python + FastAPI + boto3 | Dev D | _pendiente_ |
| 6 | `aeropuerto-frontend` | Frontend — SPA | React + Vite (deploy AWS Amplify) | Dev E | _pendiente_ |
| 7 | `aeropuerto-data-science` | Data Science — ingesta + Glue + Athena | Python (3 contenedores Docker) | Dev D + A/B/C | _pendiente_ |
| 8 | `aeropuerto-infra-deploy` | Infraestructura y despliegue | Terraform/scripts + docker compose + nginx | Lead | _pendiente_ |

## Convenciones comunes a todos los repos

- Rama `main` protegida; trabajo por ramas `feature/*`; PR con al menos 1 revisión.
- `LICENSE` (MIT) + `README.md` con instrucciones de "cómo levantar localmente" y "cómo levantar tras
  corte de sesión del Learner Lab".
- Los repos de microservicio publican su imagen a **GHCR** (`ghcr.io/btoroled/<repo>:<tag>`) mediante
  GitHub Actions al crear un tag `vX.Y`.
- Cada microservicio incluye `openapi.yaml` (contract-first) y expone Swagger-UI en `/docs`.
