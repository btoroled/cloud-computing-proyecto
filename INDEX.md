# Índice de repositorios — Proyecto Parcial CS2032 (2026-2)

Todos los repositorios son **públicos** en GitHub (verificado el 2026-09-21).

| # | Repo | Componente | Stack | Dueño | URL |
|---|---|---|---|---|---|
| 0 | `cloud-computing-proyecto` | Documentación | Markdown / draw.io | Benja | https://github.com/btoroled/cloud-computing-proyecto |
| 1 | `ms1-pasajeros-api` | Backend — MS1 Pasajeros / Tickets | Python + FastAPI + MySQL 8 | Guillermo | https://github.com/Cloud-MLA/ms1-pasajeros-api |
| 2 | `ms2-vuelos-api` | Backend — MS2 Vuelos / Operaciones | Java + Spring Boot + PostgreSQL 16 | Mariano | https://github.com/Cloud-MLA/ms2-vuelos-api |
| 3 | `ms3-infraestructura-api` | Backend — MS3 Infraestructura / Incidencias | Node.js + Express + MongoDB 7 | Edinson | https://github.com/Cloud-MLA/ms3-infraestructura-api |
| 4 | `ms4-manifiesto-api` | Backend — MS4 Manifiesto de Vuelo (sin BD) | Python + FastAPI | Fabricio | https://github.com/Cloud-MLA/ms4-manifiesto-api |
| 5 | `ms5-analitica-api` | Backend — MS5 Analítico (Athena) | Python + FastAPI + boto3 | Fabricio | https://github.com/Cloud-MLA/ms5-analitica-api |
| 6 | `aeropuerto-frontend` | Frontend — SPA | React + Vite (deploy AWS Amplify) | Alexander | https://github.com/Cloud-MLA/aeropuerto-frontend |
| 7 | `aeropuerto-data-science` | Data Science — ingesta + Glue + Athena | Python (3 contenedores Docker) | Fabricio + Guillermo/Mariano/Edinson | https://github.com/Cloud-MLA/aeropuerto-data-science |
| 8 | `aeropuerto-infra-deploy` | Infraestructura y despliegue | Terraform/scripts + docker compose + nginx | Benja | https://github.com/Cloud-MLA/aeropuerto-infra-deploy |

## Convenciones comunes a todos los repos

- Rama `main` protegida; trabajo por ramas `feature/*`; PR con al menos 1 revisión.
- `LICENSE` (MIT) + `README.md` con instrucciones de "cómo levantar localmente" y "cómo levantar tras
  corte de sesión del Learner Lab".
- Los repos de microservicio publican su imagen a **Docker Hub** (`btoroled/<repo>:<tag>`, pública, sin
  login) mediante GitHub Actions al crear un tag `vX.Y` o al hacer push a `main`. Migrado desde GHCR el
  2026-09-22 porque los paquetes de GHCR quedaban con visibilidad privada por defecto y bloqueaban el
  `docker compose pull` anónimo en VM-PROD.
- Cada microservicio incluye `openapi.yaml` (contract-first) y expone Swagger-UI en `/docs`.
