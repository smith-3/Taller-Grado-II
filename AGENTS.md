# Guía para Agentes de IA (AGENTS.md)

Este documento establece las reglas fundamentales y los modos de operación para cualquier agente de IA que trabaje en este proyecto de tesis. **Sigue estas instrucciones estrictamente.**

## 1. Regla de Oro: COMPILACIÓN CONTROLADA
Los agentes **PUEDEN** ejecutar comandos de compilación de LaTeX (como `pdflatex`, `latexmk`, etc.) bajo las siguientes condiciones estrictas:
- **SIEMPRE** se debe dirigir la salida al directorio `out/` (ej. `-output-directory=out`).
- No se deben dejar archivos auxiliares (`.aux`, `.log`, `.fls`) en la raíz del proyecto.
- El objetivo de la compilación debe ser únicamente verificar la integridad estructural y sintáctica de los cambios realizados.

## 2. Modos de Operación
Antes de realizar cualquier cambio, identifica en qué "modo" estás operando según la solicitud del usuario:

### A. Modo Análisis e Integración (Replanteamiento)
Se activa cuando el objetivo es:
- Integrar nueva información técnica o teórica.
- Reestructurar una sección completa que está mal enfocada.
- Expandir contenidos basándose en los planes de trabajo (`plan_*.md`).
- **Acción:** Requiere una lectura profunda de la estructura y reglas de contenido antes de escribir.

### B. Modo Corrección Directa (Refinado)
Se activa cuando el objetivo es:
- Corregir ortografía y acentos.
- Eliminar patrones repetitivos o muletillas.
- Cambiar palabras específicas por sinónimos más académicos.
- Ajustar puntuación y fluidez.
- **Acción:** Cambios puntuales y precisos, manteniendo el sentido original pero elevando la calidad técnica.

## 3. Uso de la Documentación (Archivos .md)
Este proyecto tiene reglas estrictas documentadas en archivos Markdown. **Debes consultarlos antes de editar archivos LaTeX.**

| Archivo | Cuándo consultarlo |
| :--- | :--- |
| **`Estructura.md`** | **CRÍTICO.** Contiene el orden de capítulos, contenido requerido y las **reglas obligatorias de figuras y prohibiciones de redacción.** |
| **`Redaccion.md`** | Para asegurar el tono formal, evitar eufemismos prohibidos, usar sinónimos recomendados y respetar el modo impersonal (3ra persona). |
| **`plan_*.md`** | Para conocer la hoja de ruta de una sección específica antes de expandir contenido. |
| **`Encuesta.md`** | Para extraer datos reales si se está trabajando en el Capítulo III (Área de Aplicación). |

## 4. Diagramas, Tablas y Recursos Visuales
Cuando se necesite explicar un flujo, proceso, arquitectura, relación de datos, decisión algorítmica o resultado tabular complejo, se debe preferir una representación visual clara antes que párrafos extensos.

### A. Diagramas Mermaid locales
- La carpeta oficial para diagramas generados es **`images/diagrams/`**.
- Todo diagrama nuevo debe conservar su fuente **`.mmd`** junto a la imagen exportada **`.png`** con el mismo nombre base.
  - Ejemplo: `images/diagrams/cap4_f3_decisiones_generador.mmd` y `images/diagrams/cap4_f3_decisiones_generador.png`.
- Para generar imágenes se debe usar Mermaid local mediante `mmdc`, aplicando la configuración existente:

```bash
npx -y @mermaid-js/mermaid-cli \
  -i images/diagrams/nombre_diagrama.mmd \
  -o images/diagrams/nombre_diagrama.png \
  -c images/diagrams/mermaid_config_large.json \
  -p images/diagrams/puppeteer-config.json
```

- Si el diagrama es muy grande, se debe dividir en partes (`part1`, `part2`, etc.) antes que reducirlo hasta volverlo ilegible.
- Los diagramas deben tener textos cortos, verbos claros y nombres consistentes con el capítulo.
- Evitar diagramas decorativos: cada figura debe explicar una decisión, flujo, componente o dato que el texto menciona.

### B. Uso en LaTeX
- Toda figura Mermaid debe insertarse desde su `.png`, no desde el `.mmd`.
- Se debe cumplir la regla de `Estructura.md`: primero se menciona la figura en el párrafo y luego se inserta.
- Usar `\caption[...] {...}` cuando el título largo pueda afectar el índice de figuras.
- Ajustar el ancho (`width`) para legibilidad real en PDF. No usar diagramas que requieran zoom excesivo.
- Mantener etiquetas descriptivas y estables, por ejemplo `\label{fig:cap4_f3_decisiones_generador}`.

### C. Tablas y diagramas de flujo
- Si se comparan alternativas, parámetros, resultados de pruebas o requisitos, usar tabla.
- Si se explica una secuencia, condición, bifurcación o proceso de usuario/sistema, usar diagrama de flujo o secuencia en Mermaid.
- Para tablas extensas, dividir por partes o mover el detalle a anexos si interrumpe la lectura principal.
- Las tablas también deben mencionarse antes de aparecer y deben mantener redacción académica, impersonal y precisa.

## 5. Flujo de Trabajo Sugerido
1. **Identificar la Tarea:** ¿Es una corrección de estilo o una reestructuración de contenido?
2. **Cargar Contexto:** Leer el archivo `.tex` objetivo Y el archivo `.md` de reglas correspondiente (mínimo `Estructura.md` y `Redaccion.md`).
3. **Verificar Prohibiciones:** Asegurarse de no violar reglas estructurales (ej. no poner figuras antes de ser mencionadas).
4. **Planificar Recursos Visuales:** Si una explicación técnica mejora con tabla, flujo o diagrama, crear/actualizar el `.mmd` en `images/diagrams/`, generar el `.png` y referenciarlo correctamente en LaTeX.
5. **Aplicar Cambios:** Realizar la edición en el archivo fuente.
6. **Reportar:** Explicar brevemente qué se cambió y bajo qué reglas se hizo.
