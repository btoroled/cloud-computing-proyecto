# Riesgos y contingencias

Proyecto Parcial CS2032 — Cloud Computing (2026-2).

| # | Riesgo | Impacto | Mitigación / contingencia | Dueño | Validar antes de |
|---|---|---|---|---|---|
| R1 | **AWS Amplify restringido/no disponible** en Learner Lab | Bloquea entregable Frontend | Contingencia: **S3 static website + CloudFront**. Probar Amplify el Día 1 | Dev E + Lead | Vie 4-Set |
| R2 | **VPC Link / API Gateway privado** no funciona con `LabRole` o exige NLB | Bloquea "APIs por API Gateway" | Usar **NLB** en vez de ALB; si VPC Link no está, documentar la limitación y exponer el ALB con IP pública **solo para la demo** | Lead | Vie 4-Set |
| R3 | **Temporizador de sesión (~4 h)** apaga las EC2 | Pérdida de tiempo por reinicios | Todo reproducible: `user-data` + `compose` + imágenes en GHCR; `RUNBOOK.md` con objetivo "<15 min"; sin estado local; dump de BD a S3 al cerrar sesión | Lead | Continuo |
| R4 | **Presupuesto (~$50–80)** | Corte de la cuenta del lab | `t3.small`/`micro`; apagar VMs fuera de sesión; 1 sola VM-INGESTA; Athena particionado por fecha; CSV con `LIMIT` en pruebas | Lead | Continuo |
| R5 | **Integridad de datos entre microservicios** | `JOIN` de Athena vacíos | Generación en orden MS2→MS1→MS3 con rangos de ID fijos; `seeds/` compartido con `SEED` fijo | Dev A/B/C | Sáb 13-Set |
| R6 | **18 días muy ajustados** | Alcance incompleto | Contract-first desde el Día 1 para paralelizar; alcance mínimo viable por servicio (solo endpoints que usan Frontend / MS4 / MS5); MS4/MS5 deliberadamente pequeños; Frontend funcional, no pulido; standup diario | Todos | Continuo |
| R7 | **Java/Spring Boot pesado en `t3.small`** | OOM en VM-PROD (5 contenedores) | `-Xmx256m` + límite de memoria del contenedor; si compite por RAM, subir VM-PROD a `t3.medium` | Dev B + Lead | Sáb 6-Set |
| R8 | **`persona` duplicada (IsA partida)** confunde al evaluador | Observación en la rúbrica de modelado | Documentar la transformación en el informe (requerimientos §3.1) con E/R por servicio | Dev D | Sáb 13-Set |
| R9 | **Crawlers de Glue infieren tipos/columnas mal** desde CSV | Consultas Athena rotas | Fijar el esquema en el `CREATE TABLE` de Glue o usar formato Parquet con esquema explícito; validar con `SELECT * LIMIT 10` por tabla | Dev D | Sáb 13-Set |
| R10 | **Integrante no entrega su parte a tiempo** | Bloqueo del camino crítico | Contratos OpenAPI + *stubs* permiten avanzar con mocks; el Lead reasigna en el standup; MS4/MS5 y Frontend pueden trabajar contra datos de ejemplo | Lead | Continuo |

## Disparadores de escalamiento

- Si al **Vie 4-Set** no hay claridad sobre Amplify **y** VPC Link → decidir contingencias ese mismo
  día y ajustar el diagrama de arquitectura.
- Si al **Mié 17-Set** falta la carga de 20 000 registros en alguna BD → priorizar sobre features
  nuevos; es requisito duro de la rúbrica.
- Si al **Vie 19-Set** el pipeline Athena no produce las 4 consultas + 2 vistas → usar datos ya en S3
  aunque no sean el 100%, dejando constancia en el informe.
