# Contrato de errores común (BE-TX-08)

Formato **único** de respuesta de error para los 5 microservicios. Objetivo: que el Frontend y MS4
puedan manejar errores de cualquier servicio con el mismo código.

Cualquier cambio se comunica en el standup y se versiona aquí.

---

## 1. Cuerpo de la respuesta de error

Todas las respuestas `4xx` y `5xx` devuelven **este JSON** con `Content-Type: application/json`:

```json
{
  "error": {
    "code": "VUELO_NO_EXISTE",
    "message": "El vuelo 99999 no existe o está cancelado.",
    "status": 422,
    "details": [
      { "field": "vuelo_id", "issue": "no encontrado en MS2" }
    ],
    "service": "ms1-pasajeros-api",
    "timestamp": "2026-09-08T14:22:00Z",
    "request_id": "c0ffee-1234"
  }
}
```

| Campo | Obligatorio | Descripción |
|---|---|---|
| `error.code` | sí | Identificador estable en `MAYUSCULA_CON_GUION_BAJO` (tabla §3). El Frontend hace `switch` sobre esto, **no** sobre el `message`. |
| `error.message` | sí | Texto legible en español, apto para mostrar al usuario. |
| `error.status` | sí | Repite el código HTTP (facilita el logging del cliente). |
| `error.details` | no | Lista de `{field, issue}` para errores de validación (422). Vacío o ausente si no aplica. |
| `error.service` | sí | Nombre del repo/imagen que originó el error. |
| `error.timestamp` | sí | ISO 8601 UTC. |
| `error.request_id` | no | Correlación entre servicios; se propaga en el header `X-Request-Id` (§4). |

---

## 2. Códigos HTTP que se usan

| HTTP | Cuándo | Ejemplo |
|---|---|---|
| **400** Bad Request | JSON malformado, tipo de dato incorrecto, parámetro de query inválido | body no parseable; `?fecha=ayer` |
| **404** Not Found | El recurso pedido por su ID no existe | `GET /tickets/999` |
| **409** Conflict | Choca con el estado actual del recurso | segundo check-in del mismo ticket; cerrar una incidencia ya cerrada |
| **422** Unprocessable Entity | El body está bien formado pero viola una **regla de negocio** o de validación | `POST /tickets` con vuelo cancelado; acople de clase mayor a manga menor; enum fuera de rango |
| **502** Bad Gateway | Un servicio del que dependemos respondió con error o formato inesperado | MS1 llama a MS2 y MS2 devuelve 500 |
| **503** Service Unavailable | Una dependencia no respondió a tiempo (timeout) o está caída | MS4 no alcanza a MS3 |
| **500** Internal Server Error | Bug no controlado. **No** debe filtrar stack traces en `message`. | excepción no manejada |

> Regla: **validación de entrada → 400**; **regla de negocio → 422**; **dependencia externa falla → 502/503**.
> `401`/`403` no se usan en este proyecto (no hay auth entre servicios; el borde es API Gateway).

---

## 3. Catálogo de `error.code`

### Comunes (cualquier servicio)

| code | HTTP | Significado |
|---|---|---|
| `VALIDACION` | 400 | Body/params mal formados o con tipo incorrecto |
| `NO_ENCONTRADO` | 404 | Recurso por ID inexistente |
| `CONFLICTO_ESTADO` | 409 | Operación incompatible con el estado actual |
| `REGLA_NEGOCIO` | 422 | Violación de regla de negocio no cubierta por un code específico |
| `ENUM_INVALIDO` | 422 | Valor fuera del [diccionario de enums](enums.md) |
| `DEPENDENCIA_ERROR` | 502 | Un servicio dependiente respondió con error |
| `DEPENDENCIA_TIMEOUT` | 503 | Un servicio dependiente no respondió a tiempo |
| `INTERNO` | 500 | Error no controlado |

### Específicos de dominio

| code | HTTP | Origen | Significado |
|---|---|---|---|
| `VUELO_NO_EXISTE` | 422 | MS1, MS3, MS4 | `vuelo_id` no existe o está `Cancelado` (según `GET /vuelos/{id}/exists` de MS2) |
| `TICKET_YA_TIENE_CHECKIN` | 409 | MS1 | Segundo check-in sobre el mismo ticket |
| `EQUIPAJE_TAG_DUPLICADO` | 409 | MS1 | `tag_id` de equipaje ya registrado |
| `TRANSICION_ESTADO_INVALIDA` | 422 | MS2 | Cambio de `estado_vuelo` que la máquina de estados no permite |
| `INCIDENCIA_YA_CERRADA` | 409 | MS3 | `PATCH /incidencias/{id}/cierre` sobre una ya cerrada |
| `ACOPLE_CLASE_INVALIDO` | 422 | MS3 | Aeronave de clase mayor a la `clase_max` de la manga (`A<B<C<D<E<F`) |
| `RECURSO_OCUPADO` | 409 | MS3 | Asignar un recurso que no está `Libre` |
| `MANIFIESTO_VUELO_NO_EXISTE` | 404 | MS4 | `GET /manifiesto/{vuelo_id}` de un vuelo inexistente |
| `ATHENA_QUERY_ERROR` | 502 | MS5 | La consulta Athena falló o el catálogo no está listo |

Añadir aquí cualquier code nuevo antes de usarlo.

---

## 4. Propagación entre servicios

- Toda petición saliente lleva el header **`X-Request-Id`** (se genera en el primer servicio; se reenvía tal cual).
- Si MS-A llama a MS-B y MS-B devuelve error:
  - MS-A responde **502** (`DEPENDENCIA_ERROR`) o **503** (`DEPENDENCIA_TIMEOUT`).
  - `error.details` incluye `{ "field": "ms2", "issue": "502 al validar vuelo" }`.
  - Se **loguea** el `request_id` y el body de error recibido (a stdout, lo recoge CloudWatch).
- MS4, al agregar, puede devolver **200 con `warnings[]`** si una dependencia **no crítica** falla
  (p. ej. MS3 caído → manifiesto sin incidencias). Las dependencias críticas (MS2 para el vuelo) sí cortan con error.

---

## 5. Implementación por stack (referencia rápida)

| Stack | Cómo | Nota |
|---|---|---|
| FastAPI (MS1, MS4, MS5) | `exception_handler` global para `HTTPException` + `RequestValidationError` → arma el JSON de §1 | `RequestValidationError` → 400 `VALIDACION` |
| Spring Boot (MS2) | `@RestControllerAdvice` con `@ExceptionHandler` por tipo | `MethodArgumentNotValidException` → 400 |
| Express (MS3) | middleware de error `(err, req, res, next)` al final de la cadena + `ajv` para 400 | `ajv` errors → `details[]` |

El esquema JSON de este contrato está en `contratos/error.schema.json` _(pendiente de agregar; por ahora esta tabla es la fuente)._
