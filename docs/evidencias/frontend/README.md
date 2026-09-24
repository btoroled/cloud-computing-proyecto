# Evidencias de frontend para el informe

Estado: nueve capturas del despliegue y la aplicación incorporadas el 23 de septiembre de 2026; faltan pruebas complementarias de algunos flujos.
Responsable: Alexander. Sección del informe: `informe/latex/secciones/05-frontend.tex`.

## Cómo incorporarlas

1. Probar con `VITE_USE_MOCKS=false` y `VITE_REAL_SERVICES` vacío para que MS1–MS5 sean reales.
2. Guardar la captura original aquí y copiar la imagen seleccionada a `informe/latex/imagenes/` con el nombre de la tabla.
3. Recompilar `informe/latex/main.tex`: la imagen reemplaza automáticamente su aviso de pendiente.
4. Registrar fecha, versión, URL, resultado y las llamadas observadas. Una captura de la pantalla no acredita por sí sola todas las llamadas de la matriz.
5. Si hacen falta varias capturas, usar sufijos y agregar figuras en la sección. No comprimir varias pruebas hasta volverlas ilegibles.

| ID | Archivo para el informe | Qué debe demostrar |
|---|---|---|
| E-5.1a | frontend-vuelos.png | Lista, detalle y nombres de aerolíneas desde MS2; guardar sus GET en Network. |
| E-5.1b | frontend-estado-vuelo.png | PATCH válido; si se prueba Despegado, horaReal devuelta por MS2. Utilizar un vuelo de prueba acordado. |
| E-5.1c | frontend-manifiesto.png | Detalle y resumen de MS4; registrar advertencias o datos ausentes, si los hay. |
| E-5.1d | frontend-ticket-checkin.png | Categorías, búsqueda/alta de pasajero, emisión y check-in con MS1 y selección de vuelo MS2. Puede requerir varias imágenes. |
| E-5.1e | frontend-infraestructura.png | GET de recursos e incidencias, PATCH de estado y POST de incidencia MS3. |
| E-5.1f | frontend-dashboard.png | Cinco indicadores reales y sus cinco respuestas MS5. Coordinar evidencia de Athena con Fabricio. |
| E-5.1g | frontend-swagger.png | Catálogo agregado y cada uno de los cinco contratos cargados. Usar imágenes adicionales cuando corresponda. |
| E-5.1h | frontend-responsive.png | Vista móvil sin desbordamiento horizontal de la página; anotar tamaño utilizado. |
| E-5.2 | frontend-network.png | Verbo, ruta y resultado de cada operación de la matriz REST; usar varias capturas o registro complementario. |
| E-5.3 | frontend-amplify.png | URL HTTPS del frontend y operación real desde ese origen. |

No mostrar contraseñas, tokens ni datos personales reales. Si se exporta un HAR, revisarlo y retirar credenciales antes de publicarlo.

## Registro de la prueba final

- Fecha y hora: 23 de septiembre de 2026, aproximadamente 21:42–21:49 (hora mostrada en las capturas).
- Commit/frontend desplegado para la corrección de MS4: `f177e0b` (merge del fix de rutas).
- URL frontend: https://main.d6qmhb5ipm8l2.amplifyapp.com/.
- URL API Gateway: https://0tmopxsjij.execute-api.us-east-1.amazonaws.com/.
- Configuración sin mocks: pendiente.
- E-5.1a: `frontend-vuelos.png` muestra lista de vuelos y detalle de AV2790; no acredita PATCH.
- E-5.1c: `frontend-manifiesto.png` muestra el manifiesto del vuelo 9, con dos pasajeros y uno con check-in, sin aviso de manifiesto parcial. Aeronave, equipaje y recursos no figuran informados; la captura no muestra Network.
- E-5.1d: `frontend-ticket-checkin.png` muestra ticket emitido y check-in completado con datos ficticios; no muestra las solicitudes de red de emisión/check-in.
- E-5.1e: `frontend-infraestructura.png` muestra inventario e incidencias; no acredita PATCH ni POST.
- E-5.1f: `frontend-dashboard.png` muestra cinco indicadores; `frontend-network.png` muestra cinco consultas de analítica con estado HTTP 200.
- E-5.1g: `frontend-swagger.png` muestra el contrato de MS1 cargado; no acredita los otros cuatro contratos.
- E-5.1h: `frontend-responsive.png` muestra Infraestructura en modo dispositivo de 682 px de ancho; no prueba todas las páginas móviles.
- E-5.3: `frontend-amplify.png` muestra implementación exitosa y dominio HTTPS; no acredita por sí sola la operación de todos los servicios.
- Siete imágenes son copias de los originales conservados en `docs/evidencias/despliegue/`; las capturas de ticket/check-in y manifiesto proceden de imágenes adjuntas por el responsable. Las nueve están también en `informe/latex/imagenes/`.
- Incidencias encontradas y resolución: la ruta duplicada `/api/manifiesto/manifiesto/{id}` se corrigió en el frontend; la consulta interna de MS4 a MS1 se encaminó por nginx para eliminar el 404 de tickets. El manifiesto del vuelo 9 cargó sin aviso de parcialidad.
- Criterio de dos métodos REST para MS4/MS5 aclarado con: pendiente.

## Validación local disponible

El 22 de septiembre de 2026, sobre el commit frontend `34fbd30`, se ejecutaron cuatro pruebas de contratos (4 correctas),
lint y build (correctos). Build reportó una advertencia de tamaño para el paquete Swagger.
Estas verificaciones locales no acreditan despliegue, CORS, respuestas reales ni ejecución de Athena.
