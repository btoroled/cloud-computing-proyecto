# Guion de la demo (EX-03 / EX-11)

Borrador. Dos versiones: **Hito 1** (≤10 min, exposición virtual con ACL) y **Hito 2 / presencial**
(~15 min). Todo se ejecuta por la **URL pública de API Gateway** (nunca por IPs internas).

Preparar antes: entorno levantado con el [`RUNBOOK.md`](https://github.com/Cloud-MLA/aeropuerto-infra-deploy/blob/main/RUNBOOK.md)
y un **video de respaldo** grabado en F2 por si el lab se corta.

---

## Hito 1 — demo corta (≤10 min)

| # | Quién | Muestra | Comando / acción |
|---|---|---|---|
| 1 | Benja | Arquitectura desplegada | Boceto `diagramas/arquitectura-solucion.md` + consola: 2 VM-PROD, ALB interno, API Gateway |
| 2 | Benja | Todo responde por HTTPS | `curl $BASE/api/vuelos/health` y `/api/pasajeros/health` |
| 3 | Mariano | MS2 vivo | `GET $BASE/api/vuelos` (lista) + `GET $BASE/api/vuelos/1/exists` |
| 4 | Guillermo | MS1 consume a MS2 | `POST $BASE/api/pasajeros/tickets` con `vuelo_id` válido → 201; con uno inexistente → 422 + log |
| 5 | Edinson | MS3 conectado | `POST $BASE/api/infra/incidencias` + `GET $BASE/api/infra/recursos?estado=Libre` |
| 6 | Fabricio | MS4 agrega | `GET $BASE/api/manifiesto/manifiesto/1` (vuelo + pasajeros + incidencias) |
| 7 | Alexander | Frontend en Amplify | Abrir la URL de Amplify → vista "Consulta de vuelo + manifiesto"; DevTools Network con ≥2 requests a MS2 |
| 8 | Benja | Avance ≥50% | Checklist de Hito 1 marcado; qué falta para Hito 2 |

**Reparto de pantalla:** Benja comparte; cada quien narra su paso.

---

## Hito 2 / presencial — demo completa (~15 min)

| # | Quién | Muestra |
|---|---|---|
| 1 | Benja | Diagrama final + recorrido de la topología (VPC, subredes, VPC Link, ALB, VM-DB privada) |
| 2 | Benja | **Seguridad:** `nc -zv <alb-dns> 80` desde fuera → falla · `mysql -h <db>` desde fuera → falla · solo API Gateway responde |
| 3 | Guillermo/Mariano/Edinson | Los 3 CRUD por la API pública + Swagger `/docs` de cada uno |
| 4 | Guillermo/Mariano/Edinson | `COUNT(*)` ≥ 20 000 en `ticket`, `vuelo`, `incidencias` |
| 5 | Fabricio | MS4 manifiesto completo + MS5: `GET /api/analitica/recaudacion-tuua-por-categoria` (con query id de Athena visible) |
| 6 | Fabricio | Data lake: `aws s3 ls .../raw/ --recursive` · Glue console · 1 consulta Athena con `JOIN` en vivo |
| 7 | Alexander | Las 4 vistas del frontend consumiendo los 5 MS; Dashboard con datos de Athena; página Swagger agregada |
| 8 | Benja | `RUNBOOK`: reinicio cronometrado (o el video) < 15 min |
| 9 | Todos | Cierre: contingencias aplicadas y limitaciones del lab |

---

## Checklist previo a la demo (Vie 19, EX-11)

- [ ] Entorno levantado y `smoke-e2e.sh` en verde
- [ ] `$BASE` (URL de API Gateway) confirmada y compartida
- [ ] Seeds cargados (≥20k) en las 3 BD
- [ ] Frontend en Amplify accesible
- [ ] Video de respaldo grabado y subido a `docs/evidencias/`
- [ ] Cada integrante ensayó su parte; tiempo total cronometrado
- [ ] Plan B si el lab se corta: reproducir con el RUNBOOK o pasar el video
