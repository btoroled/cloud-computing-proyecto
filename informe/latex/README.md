# Informe en LaTeX — CS2032

Plantilla basada en `../plantilla-informe.md`. Los textos «Pendiente» son instrucciones de redacción, no resultados verificados.

## Archivos

- `main.tex`: documento principal e índice.
- `portada.tex`: portada; completar nombres, códigos y profesor.
- `secciones/`: nueve secciones y anexo de trazabilidad.
- `imagenes/`: logo, diagramas y capturas usadas por el documento.

## Trabajo en equipo

Cada integrante edita el archivo de su sección, guarda los cambios en una rama y abre un PR. Coordinar cambios compartidos en `main.tex` y `portada.tex`. Benjamín consolida la versión final. No incluir contraseñas ni tokens en las evidencias.

## Generar el PDF en Overleaf

1. Descargar la versión acordada del repositorio.
2. Comprimir **el contenido de esta carpeta** como ZIP (main.tex debe quedar en la raíz del ZIP).
3. En Overleaf, crear un proyecto desde el ZIP. Seleccionar `main.tex` como documento principal y pdfLaTeX como compilador.
4. Recompilar y descargar el PDF. Revisar portada, índice, tablas y capturas antes de entregar.

Una persona consolida y sube a Overleaf los cambios aprobados en GitHub. La sincronización automática de GitHub con Overleaf requiere un plan que la incluya. GitHub es la fuente compartida: devolver al repositorio cualquier corrección hecha en Overleaf.

También se puede compilar localmente con `latexmk -pdf main.tex`, o ejecutar `pdflatex main.tex` dos veces para actualizar el índice.

## Imágenes y evidencias

Colocar el logo oficial en `imagenes/utec_logo.png`; mientras falte se muestra un recuadro. Copiar las imágenes que se van a incluir a `imagenes/` para que el ZIP sea independiente del resto del repositorio. Conservar las evidencias originales en `docs/evidencias/`.

Ejemplo dentro de una sección:

```tex
\begin{figure}[htbp]
  \centering
  \includegraphics[width=0.95\linewidth]{imagenes/frontend-vuelos.png}
  \caption{Consulta de vuelos conectada a la API real. Evidencia E-5.1.}
  \label{fig:frontend-vuelos}
\end{figure}
```

Solo añadir esa figura cuando exista el archivo. Referenciarla con `\ref{fig:frontend-vuelos}`. Escapar guiones bajos en texto con `\_`; para URLs usar `\url{https://...}`.

## Antes de entregar

La sección 5 contiene la redacción del frontend y la matriz de 20 operaciones. Sus capturas se insertan automáticamente cuando existen los archivos previstos; mientras falten, aparecen recuadros de evidencia pendiente. El registro y los nombres están en [la guía de evidencias](../../docs/evidencias/frontend/README.md).

La subsección `secciones/06-catalogo-er.tex` incluye el diagrama propuesto y explica las 19 tablas y sus uniones. La imagen está incluida en `imagenes/`, por lo que viaja en el ZIP. Queda contrastar el modelo con Glue desplegado.

Completar los datos de la portada, reemplazar los pendientes por contenido comprobado, incorporar las capturas reales, completar la trazabilidad y revisar el PDF generado. La plantilla no da por terminada ninguna integración.
