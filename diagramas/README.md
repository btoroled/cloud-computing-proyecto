# Diagrama de arquitectura de solución

- [`arquitectura-solucion.md`](arquitectura-solucion.md) — **boceto v0** (Mermaid, DA-01). Se mantiene al día en F0/F1.
- [`arquitectura-solucion.drawio`](arquitectura-solucion.drawio) + [`arquitectura-solucion-v2.png`](arquitectura-solucion-v2.png)
  — **versión final (DA-03)**, generada el 2026-09-22 a partir de `aeropuerto-infra-deploy/terraform/` +
  `RUNBOOK.md` (fuente de verdad de nombres de recursos/SG/subredes). Referenciada en
  `informe/latex/secciones/02-arquitectura.tex`. Abrir el `.drawio` en [app.diagrams.net](https://app.diagrams.net)
  para editar.
- **Muestra el estado objetivo** (VM-PROD sin IP pública, solo detrás del ALB — tras cerrar `sg-vm-prod`
  en BE-INT-06). Si al desplegar hoy la infra real termina difiriendo (otros CIDR, otra AMI, SG
  distintos), **regenerar antes de exportar el PDF final** — pendiente de **DA-04** (revisión cruzada
  con Fabricio/Alexander) y confirmación con la consola AWS real.
- [`er-catalogo-datalake.drawio`](er-catalogo-datalake.drawio) + [`PNG`](er-catalogo-datalake.png) — modelo E/R propuesto del catálogo `aeropuerto_lake` (DS-13): 19 tablas de MS1, MS2 y MS3, con referencias suaves entre fuentes.

Debe incluir todos los servicios AWS de Backend + Frontend + Data Science.
Ver la topología y la lista de servicios en [`../docs/arquitectura.md`](../docs/arquitectura.md).
