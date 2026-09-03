# Hallazgos del AWS Academy Learner Lab

Se completa en la **Fase 0** (antes del Sáb 6-Set). Dueño: Lead.

Objetivo: saber qué servicios están disponibles con `LabRole` y con qué límites, antes de comprometer
la arquitectura.

## Checklist de validación (en este orden)

| # | Servicio / capacidad | ¿Disponible? | Límites / notas | Fecha |
|---|---|---|---|---|
| 1 | EC2: crear VPC, subredes, IGW, NAT, SG | ⬜ | | |
| 1 | EC2: lanzar instancias `t3.small` / `t3.medium` con `LabInstanceProfile` | ⬜ | | |
| 1 | SSM Session Manager para acceso sin puerto 22 | ⬜ | | |
| 2 | S3: crear bucket, subir/leer objetos | ⬜ | | |
| 3 | AWS Glue: crear base de datos y crawlers con `LabRole` | ⬜ | | |
| 3 | Amazon Athena: ejecutar consultas + crear vistas (workgroup, output en S3) | ⬜ | | |
| 4 | API Gateway: crear HTTP API | ⬜ | | |
| 4 | **VPC Link** para integración privada con ALB/NLB | ⬜ | ¿exige NLB? ¿`LabRole` alcanza? | |
| 5 | **AWS Amplify**: crear app y desplegar | ⬜ | si **no** → contingencia S3 + CloudFront | |
| 5 | CloudFront + S3 static website (contingencia de Amplify) | ⬜ | | |
| 6 | CloudWatch Logs desde las EC2 | ⬜ | | |
| 6 | Presupuesto restante de la cuenta | ⬜ | $ | |

## Decisiones tomadas a partir de los hallazgos

_(pendiente)_

- Balanceador: ALB interno / NLB interno → …
- Frontend: Amplify / S3+CloudFront → …
- Exposición HTTPS: API Gateway + VPC Link / alternativa → …
