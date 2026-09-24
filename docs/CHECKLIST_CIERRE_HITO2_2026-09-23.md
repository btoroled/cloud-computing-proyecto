# Checklist actualizado de cierre — Hito 2

**Corte:** 23 de septiembre de 2026, 22:08 (Lima). **Entrega indicada en el README del proyecto:** 23:59 en Canvas. Este documento distingue lo comprobado de lo que todavía necesita una prueba o evidencia; no convierte una tarea en terminada solo porque la pantalla abra.

## 1. Ya comprobado: no repetir estos arreglos esta noche

- [x] Frontend publicado en `https://main.d6qmhb5ipm8l2.amplifyapp.com/` y conectado a `https://0tmopxsjij.execute-api.us-east-1.amazonaws.com`.
- [x] La regla SPA de Amplify cambió de `404-200` a `200`: `/dashboard/`, `/tickets/`, `/infraestructura/` y `/docs/` respondieron HTTP 200 al abrirse directamente. El paso de reemplazar la regla quedó en el README del frontend (PR #16).
- [x] CORS y preflight de API Gateway/Nginx quedaron incorporados al repositorio de infraestructura; no hay que editar los contenedores a mano en cada reinicio normal.
- [x] Fix de fechas de Athena de MS5 mergeado; imagen nueva instalada en ambas VM-PROD. Las cinco rutas de Analítica devolvieron HTTP 200 y el dashboard mostró sus cinco tarjetas.
- [x] Mejoras de navegación/listados e incidencias del frontend mergeadas (PR #14 y #15).
- [x] Comprobación pública de solo lectura: categorías de MS1, aerolíneas de MS2, recursos de MS3 y los cinco documentos OpenAPI devolvieron HTTP 200.
- [x] PPT de Hito 2 existe en el repositorio central; **que exista no sustituye su revisión final**.

## 2. Prioridad P0: entregar antes de las 23:59

Marca cada casilla únicamente al tener el resultado y, cuando aplique, su archivo o enlace de evidencia.

- [ ] **Asignar dueño de la subida a Canvas (Benja/equipo)** y confirmar quién tendrá el PDF, PPT y enlaces finales antes de las 23:40. Cierre: una persona confirma explícitamente que hará la entrega.
- [ ] **Completar portada (Benja/equipo):** apellidos y códigos faltantes, nombre del profesor y logo UTEC o una decisión explícita de omitirlo. En `informe/latex/portada.tex` de `main` todavía hay marcadores `[pendiente]`.
- [x] **Capturar y guardar evidencia frontend (Alexander):** Dashboard con cinco tarjetas y Network con cinco GET 200; Operaciones con vuelo/detalle, APIs con una especificación cargada, Infraestructura, móvil y ticket/check-in. Las capturas están en `docs/evidencias/frontend/` y sus copias seleccionadas en `informe/latex/imagenes/`.
- [ ] **Probar los flujos de escritura que aún no están acreditados (Alexander + dueños de MS):** ticket/check-in, cambio de estado de un vuelo de prueba, y cambio de recurso/creación de incidencia. No alterar registros reales sin coordinar; si un flujo no se puede probar, declararlo como pendiente en vez de inventar un éxito.
- [ ] **Probar manifiesto MS4 con un vuelo existente (Alexander/Fabricio):** el frontend desplegado pidió `/api/manifiesto/manifiesto/4` y recibió 404; OpenAPI publica `/api/manifiesto/{vuelo_id}`. El frontend está corregido localmente, pero necesita despliegue y nueva prueba de manifiesto y resumen.
- [ ] **Acreditar Athena (Fabricio):** adjuntar captura de consola o identificadores/resultados de las cinco consultas exitosas, y confirmar de qué tablas/ingestas provienen. Los HTTP 200 del frontend prueban integración, no por sí solos toda la procedencia de datos.
- [ ] **Integrar las capturas al LaTeX (Alexander/Benja):** copiar las seleccionadas a `informe/latex/imagenes/` con los nombres que ya referencia `secciones/05-frontend.tex`; actualizar texto, URLs y estado de prueba. No dejar una leyenda que diga «pendiente» junto a una prueba ya terminada.
- [ ] **Cerrar secciones y conclusiones (Benja + dueños):** revisar los `\pendiente{...}` restantes, en especial arquitectura, MS3/MS4, Athena, recuperación tras corte y conclusiones. Resolver lo que tenga evidencia y declarar honestamente lo que no alcance.
- [ ] **Compilar y revisar PDF final en Overleaf (Benja):** subir como ZIP el contenido de `informe/latex/`, usar `main.tex` y pdfLaTeX; revisar portada, índice, figuras, tablas, URLs y páginas. GitHub no tiene autocompilación LaTeX configurada.
- [ ] **Revisar PPT y subir entrega (Benja/equipo):** comprobar que las diapositivas no contradigan el PDF ni afirmen pruebas no realizadas; subir PDF, PPT y enlaces a Canvas. Guardar confirmación de envío antes de las 23:59.

## 3. Matriz rápida de pruebas y evidencias

| Área | Estado que sí conocemos | Falta para poder marcar «completo» |
| --- | --- | --- |
| Operaciones / MS2 | Tabla y detalle de vuelo capturados. | Transición `PATCH` válida de un registro de prueba. |
| Tickets / MS1 | Ticket emitido y check-in completado con datos ficticios; captura guardada. | Captura Network de emisión/check-in, si se exige acreditación de llamadas. |
| Infraestructura / MS3 | Inventario e incidencias visibles; captura guardada. | `PATCH` de recurso y `POST` de incidencia de prueba con persistencia. |
| Manifiesto / MS4 | OpenAPI muestra la ruta correcta; frontend desplegado pide ruta duplicada y recibe 404. | Desplegar fix y comprobar manifiesto y resumen en UI/Network. |
| Analítica / MS5 | Cinco indicadores y cinco GET 200 capturados. | Evidencia de consultas Athena/procedencia. |
| APIs / Swagger | Catálogo y contrato MS1 cargado en captura. | Confirmar visualmente los otros cuatro contratos si se exige. |
| Responsive | Infraestructura capturada en vista móvil de 682 px. | Revisar otras vistas móviles si se exige. |

**Archivos de captura esperados por `05-frontend.tex`:** `frontend-vuelos.png`, `frontend-estado-vuelo.png`, `frontend-manifiesto.png`, `frontend-ticket-checkin.png`, `frontend-infraestructura.png`, `frontend-dashboard.png`, `frontend-swagger.png`, `frontend-responsive.png`, `frontend-network.png` y `frontend-amplify.png`. No crear una imagen ficticia para llenar un espacio: si falta una prueba, corregir el texto del informe.

## 4. Verificación de equipo y documentación

- [ ] Confirmar con Edinson/Fabricio que la ingesta MS3 mergeada realmente corrió en la cuenta actual; el merge de código no prueba ejecución de producción.
- [ ] Confirmar con Fabricio el catálogo Glue, archivos S3 y las consultas Athena; contrastar el diagrama E/R del lago con lo desplegado.
- [ ] Confirmar con responsables de MS1/MS2/MS3 sus conteos y evidencias más recientes y que las rutas del informe coincidan con el despliegue.
- [ ] Revisar los nueve enlaces de `INDEX.md` y la URL pública de Amplify/API justo antes de exportar el PDF.
- [x] Revisar evidencia local en `docs/evidencias/despliegue/` antes de añadirla: se seleccionaron las capturas pertinentes y se conservaron los originales.
- [ ] Evitar capturas con contraseñas, tokens, datos personales reales o identificadores de sesión visibles.

## 5. Orden de ejecución sugerido hasta medianoche

| Franja aproximada (Lima) | Objetivo |
| --- | --- |
| 22:10–22:35 | Publicar el fix de MS4, verificar manifiesto y obtener su captura si responde. |
| 22:35–23:10 | Actualizar sección frontend y cerrar marcadores críticos del informe. |
| 23:10–23:35 | Compilar/revisar PDF en Overleaf y revisar PPT. |
| 23:35–23:50 | Subir a Canvas y conservar margen para un error de última hora. |

**No bloquear la entrega por mejoras opcionales:** no rehacer Terraform, no cambiar arquitectura a última hora y no esperar una captura perfecta de un flujo que siga fallando. Documentar con precisión el resultado real y priorizar un PDF entregado a tiempo.
