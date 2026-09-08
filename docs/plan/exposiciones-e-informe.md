# Plan de trabajo — Exposiciones e Informe

**Rúbrica:** Exposición virtual con ACL **3 pts** (Hito 1) · Exposición presencial + demo **1 pt**
(Semana 7) · el **Informe (Word/PDF)** y el **PPT** son el vehículo de todo el **Hito 2 (17 pts)**.

> La exposición **presencial es obligatoria**: si el grupo no se presenta, la evaluación es
> desaprobatoria con nota máxima 10.

Documentos base: [plan general](../plan-de-trabajo.md) · [verificación](../verificacion.md).

---

## 1. Entregables y Definición de Terminado

- [ ] **Hito 1 — exposición virtual con ACL:** slides de avance (≥50% por parte) + demo corta grabable.
      Fecha: la coordina el ACL, tope **Sáb 12-Set 23:59**.
- [ ] **Hito 2 — subida a Canvas (tope Dom 20-Set 23:59):**
  - [ ] **Informe** en Word o PDF con evidencia de **todo** lo solicitado.
  - [ ] **Resumen en PowerPoint** con todo lo solicitado.
  - [ ] `INDEX.md` con enlaces a **todos los repos públicos**.
- [ ] **Semana 7 — exposición presencial + demo en vivo** por la URL pública de API Gateway.

**DoD del informe:** cada ítem del [checklist de Hito 2](../plan-de-trabajo.md#6-checklist-hito-2--entregables-finales)
tiene su evidencia (captura/salida) referenciada · sin secciones "TODO" · enlaces verificados · PDF exportado.

---

## 2. Asignación

| Trabajo | Responsable | Aporta |
|---|---|---|
| Coordinación y consolidación del informe y el PPT | **Benja** | — |
| Sección Backend + su evidencia | Guillermo (MS1), Mariano (MS2), Edinson (MS3), Fabricio (MS4/MS5) | cada quien su microservicio |
| Sección Transformaciones del modelo (Persona partida, FK suaves) | Fabricio | — |
| Sección Frontend + matriz de cobertura REST | Alexander | — |
| Sección Data Science (ingesta, Glue, Athena, E/R catálogo) | Fabricio | Guillermo/Mariano/Edinson (su contenedor) |
| Sección Arquitectura + diagrama | Benja | — |
| Guion y ensayo de la demo en vivo | Benja | todos |
| Slides de Hito 1 (avance) | Benja consolida | cada quien 1–2 slides de su parte |

---

## 3. Estructura del informe

| # | Sección | Responsable | Evidencia principal |
|---|---|---|---|
| 1 | Introducción y objetivos | Benja | — |
| 2 | Arquitectura de solución | Benja | diagrama `draw.io` + lista de servicios AWS |
| 3 | Backend — 5 microservicios | Guillermo/Mariano/Edinson/Fabricio | Swagger de las 5 · `COUNT(*)` ≥ 20 000 en `ticket`/`vuelo`/`incidencias` · log de consumo entre servicios |
| 3b | E/R por BD SQL + JSON Schema de la NoSQL | Guillermo/Mariano/Edinson | 2 diagramas E/R + `docs/er/ms3-mongo-schemas.md` |
| 4 | Transformaciones del modelo monolítico | Fabricio | E/R por servicio; explicación de la jerarquía `Persona` y las FK suaves |
| 5 | Frontend | Alexander | capturas de las 4 vistas + panel Network con ≥2 métodos por MS · URL de Amplify |
| 6 | Data Science | Fabricio | `aws s3 ls` · Glue console · E/R del catálogo · 4 consultas Athena con resultados · 2 vistas |
| 7 | Despliegue y seguridad | Benja | `docker compose ps` ×2 · `nc`/`mysql` fallando desde fuera · URL pública HTTPS respondiendo · `RUNBOOK.md` |
| 8 | Enlaces a repositorios | Benja | `INDEX.md` |
| 9 | Conclusiones y limitaciones | Benja | contingencias aplicadas (Amplify, VPC Link, etc.) |

---

## 4. Tareas

| ID | Tarea | Resp. | Est. | Fase | Depende de | DoD |
|---|---|---|---|---|---|---|
| EX-01 | Plantilla del informe (Word/Docs) con las 9 secciones y placeholders de evidencia | Benja | 0.5d | F1 | — | Plantilla compartida |
| EX-02 | Slides de avance Hito 1 (1–2 por integrante) + consolidación | Benja + todos | 1d | F1 | avance F1 | PPT de avance listo |
| EX-03 | Guion de la demo corta de Hito 1 + ensayo interno | Benja | 0.5d | F1 | deploy v1 | Demo de ≤10 min ensayada |
| EX-04 | **Exposición virtual con ACL** | Todos | 0.5d | F1 | EX-02/03 | Realizada; feedback del ACL anotado |
| EX-05 | Carpeta `docs/evidencias/` poblada a medida que cada parte termina | Cada dev | 1d (repartido) | F2 | verificación | Todas las evidencias del checklist H2 presentes |
| EX-06 | Redacción de cada sección del informe (en paralelo) | Cada dev | 0.5d c/u | F2 | EX-01, EX-05 | Sección completa, sin "TODO" |
| EX-07 | PPT resumen del Hito 2 (estructura por Benja, 2–3 slides por integrante) | Benja + todos | 1d | F2/F3 | EX-06 | PPT completo |
| EX-08 | Consolidación final del informe + revisión cruzada + export **PDF** | Benja | 1d | F3 | EX-06 | PDF final revisado por ≥2 personas |
| EX-09 | Verificar `INDEX.md` (todos los repos públicos y accesibles) | Benja | 0.25d | F3 | repos | Enlaces abren sin login |
| EX-10 | **Subida a Canvas** (informe PDF + PPT + enlaces) antes del Dom 20-Set 23:59 | Benja | 0.25d | F3 | EX-07/08/09 | Entrega confirmada en Canvas |
| EX-11 | Ensayo general de la demo en vivo (Vie 19) + reparto de quién muestra qué | Todos | 0.5d | F2 | code freeze | Demo de ~15 min cronometrada |
| EX-12 | **Exposición presencial + demo** | Todos | 0.5d | F3 | EX-11 | Realizada |

---

## 5. Mapa a hitos

| Hito | Fecha | Qué se entrega aquí |
|---|---|---|
| **Hito 1** | Sáb 12-Set | EX-02, EX-03, **EX-04** (exposición virtual con ACL, 3 pts) |
| **Hito 2** | Dom 20-Set 23:59 | **EX-08** informe PDF + **EX-07** PPT + **EX-09** `INDEX.md` → **EX-10** subida a Canvas |
| **Semana 7** | por publicar | **EX-12** exposición presencial + demo (1 pt, obligatoria) |

---

## 6. Dependencias y riesgos

| Riesgo | Mitigación |
|---|---|
| Dejar el informe para el final | Redacción **por secciones desde F2** (EX-06), a medida que cada parte termina; el informe crece incrementalmente |
| Evidencias faltantes el día de la entrega | `docs/evidencias/` se puebla en F2 (EX-05), no en F3; checklist de [verificación](../verificacion.md) como control |
| **No presentarse a la presencial** = desaprobado | EX-11 ensayo obligatorio el Vie 19; confirmar asistencia de los 6 con anticipación |
| Demo falla en vivo (corte de sesión del lab) | Levantar el entorno con el `RUNBOOK.md` antes; tener video de respaldo de la demo grabado en F2 |
| Feedback del ACL en Hito 1 llega tarde para F2 | Anotar en EX-04 y priorizar los ajustes al inicio de F2 |
