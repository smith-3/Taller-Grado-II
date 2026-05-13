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
| **`@Estructura.md`** | **CRÍTICO.** Contiene prohibiciones específicas (ej. no mencionar "TecnoTime" en el Marco Teórico) y reglas de ubicación de figuras. |
| **`Estructura.md`** | Para entender qué debe contener cada capítulo y el orden jerárquico del documento. |
| **`Redaccion.md`** | Para asegurar el tono formal, evitar eufemismos prohibidos, usar sinónimos recomendados y respetar el modo impersonal (3ra persona). |
| **`plan_*.md`** | Para conocer la hoja de ruta de una sección específica antes de expandir contenido. |
| **`Encuesta.md`** | Para extraer datos reales si se está trabajando en el Capítulo III (Área de Aplicación). |

## 4. Flujo de Trabajo Sugerido
1. **Identificar la Tarea:** ¿Es una corrección de estilo o una reestructuración de contenido?
2. **Cargar Contexto:** Leer el archivo `.tex` objetivo Y el archivo `.md` de reglas correspondiente (mínimo `@Estructura.md` y `Redaccion.md`).
3. **Verificar Prohibiciones:** Asegurarse de no violar reglas estructurales (ej. no poner figuras antes de ser mencionadas).
4. **Aplicar Cambios:** Realizar la edición en el archivo fuente.
5. **Reportar:** Explicar brevemente qué se cambió y bajo qué reglas se hizo.
