# cloud-computing-proyecto

Repositorio de **documentación** del Proyecto Parcial de CS2032 — Cloud Computing (Ciclo 2026-2).
Universidad de Ingeniería y Tecnología (UTEC).

Dominio de negocio: **Aeropuerto Internacional Jorge Chávez**, reutilizando el modelo de datos del
proyecto de Base de Datos I y descomponiéndolo en 5 microservicios sobre AWS.

> Este repo **no tiene código**. El código vive en un repo por microservicio + frontend + data science
> + infraestructura. Ver [`INDEX.md`](INDEX.md).

---

## Fechas clave (2026)

| Hito | Fecha | Qué se entrega |
|---|---|---|
| **Hito 1 — exposición virtual con ACL** | Sáb **12-Set** 23:59 | Avance ≥50% por parte + slides + demo corta |
| **Hito 2 — entrega en Canvas** | Dom **20-Set** 23:59 | Informe (Word/PDF) + PPT + enlaces a repos |
| **Exposición presencial + demo** | Semana 7 | Demo en vivo — **obligatoria** (si no, nota máx. 10) |

---

## Mapa de la documentación

```
README.md ................ estás aquí — cómo funciona todo
INDEX.md ................. lista de los 9 repos del proyecto (URLs, dueños)

docs/
  requerimientos.md ...... QUÉ hay que construir: alcance/rúbrica, los 5 microservicios,
                           transformaciones del modelo, requisitos funcionales y no funcionales
  arquitectura.md ........ CÓMO se despliega: topología AWS, red, seguridad, servicios usados
  plan-de-trabajo.md ..... CUÁNDO y QUIÉN: repos, equipo, cronograma día a día, checklists de hito
  riesgos.md ............. 10 riesgos con contingencia y disparadores de escalamiento
  verificacion.md ....... cómo PROBAR cada ítem de la rúbrica end-to-end
  aws-learner-lab-hallazgos.md .. qué servicios están disponibles en el lab (se llena en Fase 0)

  fuentes/               documentos originales (enunciado del curso + modelo de datos del aeropuerto)

  plan/                    plan detallado por sección (tareas con ID, responsable, estimación, DoD)
    README.md ............ índice y orden de lectura de los planes
    distribucion-trabajo.md .. reparto global: RACI, carga por persona, dependencias entre personas
    backend.md ........... 7 pts — MS1..MS5 + transversal + integración/despliegue
    frontend.md ......... 3 pts — 4 vistas + matriz de cobertura REST + Amplify
    data-science.md ..... 5 pts — seeds, ingesta, Glue, 5 consultas Athena + 2 vistas
    diagrama-arquitectura.md .. 1 pt — contenido del draw.io
    exposiciones-e-informe.md . 3+1 pts — ACL, informe, PPT, presencial

  contratos/
    enums.md ............ diccionario de enums compartido por los 3 servicios con BD (OBLIGATORIO respetar)

  er/                     modelos de datos por microservicio (los suben los dueños de cada MS)
    README.md ........... qué archivo debe dejar cada quien

  evidencias/            capturas y salidas que prueban cada requisito (se llena en Fase 2)
    athena/ ............. resultados de las consultas y vistas

diagramas/ ............... arquitectura-solucion.drawio + E/R del data lake
informe/ ................ informe final Word/PDF (se arma en Fase 2-3)
ppt/ .................... resumen en PowerPoint
```

---

## Empieza aquí (todo dev, ~20 min)

Lee en este orden:

1. [`docs/requerimientos.md`](docs/requerimientos.md) — entiende el alcance completo y por qué el
   modelo del aeropuerto se parte en 5 microservicios (§2 y §3 son clave).
2. [`docs/arquitectura.md`](docs/arquitectura.md) — cómo se conecta todo en AWS (el diagrama de la §1).
3. [`docs/plan/distribucion-trabajo.md`](docs/plan/distribucion-trabajo.md) — qué repo lideras, tu
   carga por fase y de quién dependes / quién depende de ti.
4. El plan de **tu sección** en [`docs/plan/`](docs/plan/) — tus tareas con su Definición de Terminado.
5. [`docs/contratos/enums.md`](docs/contratos/enums.md) — valores que **no** puedes cambiar por tu cuenta.
6. [`docs/plan-de-trabajo.md`](docs/plan-de-trabajo.md) §4 — el cronograma día a día que ata todas las
   secciones, y §5/§6 — los checklists de Hito 1 y Hito 2.

---

## Guía por rol

| Rol | Lee y sigue | Mantiene / entrega en este repo |
|---|---|---|
| **Lead / Arquitecto** | todo | `arquitectura.md`, `diagramas/`, `aws-learner-lab-hallazgos.md`, `INDEX.md`, consolidación del `informe/` y `ppt/` |
| **Dev A** (MS1) | `plan/backend.md` §4, `contratos/enums.md` | `docs/er/ms1-mysql-er.*`, evidencia de MS1 en `docs/evidencias/backend/`, su sección del informe |
| **Dev B** (MS2) | `plan/backend.md` §5 | `docs/er/ms2-postgres-er.*`, evidencia de MS2, su sección del informe |
| **Dev C** (MS3) | `plan/backend.md` §6 | `docs/er/ms3-mongo-schemas.md`, evidencia de MS3, su sección del informe |
| **Dev D** (MS4/MS5 + Data Science) | `plan/backend.md` §7-§8, `plan/data-science.md` | `diagramas/er-catalogo-datalake.*` (o Dev E), `docs/evidencias/athena/`, secciones "Transformaciones" y "Data Science" del informe |
| **Dev E** (Frontend) | `plan/frontend.md` | matriz de cobertura REST + capturas en `docs/evidencias/frontend/`, su sección del informe, página Swagger-UI agregada |

---

## Cómo contribuir a la documentación

1. **Rama + PR.** No commitees directo a `main`. Rama `docs/<tema>`, PR con 1 revisión (normalmente el Lead).
2. **Español**, Markdown, líneas ≤ ~100 caracteres, tablas para listas de tareas/datos.
3. **Dónde va cada cosa:**
   - Diagrama E/R de tu BD → `docs/er/` (PNG + fuente `.drawio`/`.dbml`). Actualiza la tabla de
     [`docs/er/README.md`](docs/er/README.md).
   - Estructuras JSON de MongoDB → `docs/er/ms3-mongo-schemas.md`.
   - Capturas que prueban un requisito → `docs/evidencias/<área>/` con nombre descriptivo
     (`ms1-count-tickets.png`, `athena-q3-combustible.png`). El checklist está en
     [`docs/verificacion.md`](docs/verificacion.md).
   - URL de tu repo de código, una vez creado → [`INDEX.md`](INDEX.md).
   - Hallazgo sobre el Learner Lab (servicio disponible/no) → [`docs/aws-learner-lab-hallazgos.md`](docs/aws-learner-lab-hallazgos.md).
   - Texto de tu sección del informe → `informe/` (o el doc colaborativo que indique el Lead).
4. **No dupliques.** Si un dato ya está en `requerimientos.md` o `arquitectura.md`, enlázalo, no lo copies.
5. **Contratos.** Cambiar un valor de [`docs/contratos/enums.md`](docs/contratos/enums.md) o un endpoint
   compartido se avisa en el standup y se hace por PR — rompe a otros servicios y a los joins de Athena.
6. **Marca el progreso.** Al cerrar una tarea, marca su casilla en el plan de la sección y en los
   checklists de hito de [`docs/plan-de-trabajo.md`](docs/plan-de-trabajo.md).

---

## Cómo evoluciona la documentación durante el proyecto

| Fase | Fechas | Qué se actualiza aquí |
|---|---|---|
| **F0** Setup / contratos | Mié 2 – Vie 4 | `aws-learner-lab-hallazgos.md`, `contratos/enums.md`, `INDEX.md` (repos creados), `openapi.yaml` en cada repo de código |
| **F1** Núcleo + deploy v1 | Sáb 6 – Sáb 12 | Marcar checklist Hito 1; primeras entradas de `docs/er/`; slides de avance |
| **F2** Completar + endurecer | Sáb 13 – Vie 19 | `docs/evidencias/` (todo), `docs/er/` completo, `diagramas/` v2, redacción de `informe/` por secciones |
| **F3** Entrega + exposición | Sáb 20 / Sem 7 | Informe consolidado a PDF, `ppt/` final, `INDEX.md` verificado, subida a Canvas |

El **informe final no se escribe al final**: cada dev redacta su sección en F2 a medida que termina su
componente y sube las evidencias. En F3 el Lead solo consolida y exporta.

---

## Convenciones del proyecto

- **3 lenguajes / 3 BD:** MS1 Python + MySQL · MS2 Java + PostgreSQL · MS3 Node + MongoDB · MS4 Python
  sin BD · MS5 Python + Athena.
- **Enums idénticos** en los 3 servicios y en los archivos de ingesta (ver `contratos/enums.md`).
- **IDs deterministas** para la data ficticia: orden de generación MS2 → MS1 → MS3, `SEED` fijo
  (ver [`docs/plan-de-trabajo.md`](docs/plan-de-trabajo.md) §3).
- **Todo reproducible** ante el corte de sesión del Learner Lab: `docker compose` + imágenes en GHCR +
  `RUNBOOK.md`.
- Repos de código: `main` protegida, `feature/*`, PR con 1 revisión, tag `vX.Y` publica imagen a GHCR.
