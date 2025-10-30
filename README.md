# Guía para la Elaboración de Trabajos de Grado

## Universidad Mayor de San Simón
## Facultad de Ciencias y Tecnología
## Carreras de: Ingeniería Informática e Ingeniería de Sistemas

**Lic. Mgr. Rolando Jaldín Rosales**  
**Lic. Mgr. Yony Montoya Burgos**  
Cochabamba, Febrero de 2013

## Contenido
1. [OBJETIVO](#objetivo)
2. [FORMATO DE LOS TRABAJOS DE GRADO](#formato-de-los-trabajos-de-grado)
3. [NUMERO DE HOJAS E IMPRESIÓN](#numero-de-hojas-e-impresión)
4. [CONTENIDO](#contenido)
5. [PRESENTACIÓN](#presentación)
6. [DVD TRABAJO DE GRADO](#dvd-trabajo-de-grado)

## 1. OBJETIVO
La presente guía tiene el propósito de dar los lineamientos a los estudiantes de la Carrera de Ingeniería Informática de la Facultad de Ciencias y Tecnología para la elaboración de los documentos de sus trabajos de grado, sean estos: Proyectos de Grado, Trabajos de Adscripción o Trabajos Dirigidos.

Por otra parte, está el establecer un formato base de documento que reduzca sustancialmente los voluminosos documentos impresos y que son dificultosos en su revisión y su almacenamiento posterior.

## 2. FORMATO DE LOS TRABAJOS DE GRADO

### Formato de Títulos y subtítulos
- **Fuente**: Arial
- **Tamaño de la fuente**: 12 para títulos (solo letras mayúsculas), 11 para subtítulos
- **Estilo**: Negrilla
- **Color**: Negro o Azul
- **Interlineado**: Sencillo
- **Espaciado entre párrafos**: Anterior 6 pto y posterior 6 pto
- **Alineado**: Centrado para títulos, a la izquierda para subtítulos

### Formato de párrafo
- **Fuente**: Arial
- **Tamaño de la fuente**: 11
- **Estilo**: Normal
- **Color**: Negro
- **Interlineado**: Sencillo
- **Espaciado entre párrafos**: Posterior 6 pto
- **Alineado**: A ambos márgenes
- **Sangría**: Sin uso

### Formato de página
- **Tamaño**: Carta (21.59 cm x 27.94 cm)
- **Márgenes**: Izquierdo 3 cm, derecho 2 cm, superior 2 cm, inferior 2 cm
- **Numeración**: En el pié de página, centrado

### Formato de figuras
- Toda figura o cuadro debe indicar en la parte inferior del recuadro: el número, la descripción y la fuente (Ej. Figura 5. Diagrama de Casos de Uso. Elaboración propia).
- La figura deberá estar centrada y con recuadro.

## 3. NUMERO DE HOJAS E IMPRESIÓN
- El número total de hojas se establece en máximo 80 hojas, aproximadamente 160 páginas, contándose desde la carátula del documento.
- La impresión de los documentos deberán ser hechos en anverso y reverso.
- La información referenciada en el documento como ser: planos, diagramas, esquemas, imágenes, cuestionarios u otros, podrán ser almacenados en un DVD que estará anexado en la tapa posterior del documento.

## 4. CONTENIDO
Los Trabajos de Grado deberán tener la siguiente estructura general:
- Carátula del documento, 1 hoja.
- Dedicatoria y agradecimientos, 1 hoja.
- Índice de contenido, índice de figuras y cuadros, 1 hoja.
- Ficha resumen del trabajo, 1 hoja.
- Capítulo I: Introducción, máximo 6 páginas (3 hojas).
- Capítulo II: Marco Teórico o Referencial, máximo 10 páginas (5 hojas).
- Capítulo III: Área de aplicación, máximo 6 páginas (3 hojas).
- Capítulo IV: Metodología o Desarrollo del Proyecto, máximo 114 páginas (57 hojas).
- Capítulo V: Conclusiones y Recomendaciones, máximo 2 páginas (1 hoja).
- Bibliografía, máximo 2 páginas (1 hoja). Deberá usarse el estilo APA para referencias.
- Anexos, máximo 12 páginas (6 hojas).
- DVD de Anexos (opcional): planos, diagramas, fotos, imágenes, esquemas, cuestionarios, código fuente, pruebas, etc.

## 5. PRESENTACIÓN
Los documentos presentados deberán ser empastados en color azul con el mismo diseño de la carátula del documento en la tapa, según formato establecido en la FCyT.

La carátula deberá contener:
- Logo de la UMSS en la parte superior izquierda
- Logo de la Facultad de Ciencias y Tecnología en la parte superior derecha
- Nombre de la Universidad en la parte superior central, debajo el nombre de la facultad, debajo el nombre de la carrera.
- En la parte central el título del trabajo de grado.
- Descripción de la modalidad de titulación.
- Presentado por: Nombre completo del estudiante.
- Tutor: Nombre del tutor indicando su grado académico.
- Lugar, por defecto Cochabamba - Bolivia
- Mes y año de la presentación

## 6. DVD TRABAJO DE GRADO
El DVD de Trabajo de Grado es un requisito del área de Kardex de la Facultad y que es elaborado por el estudiante para su Defensa Pública, este requisito se mantiene según el formato establecido en la Facultad de Ciencias y Tecnología.

---

## Instrucciones para el Proyecto LaTeX

Este proyecto está configurado para compilar una tesis usando LaTeX. Asegúrate de tener instalado un compilador LaTeX como TeX Live o MiKTeX.

### Compilación
Para compilar el documento principal `thesis.tex`, ejecuta:
```bash
pdflatex thesis.tex
bibtex thesis
pdflatex thesis.tex
pdflatex thesis.tex
```

### Estructura del Proyecto
- `thesis.tex`: Archivo principal de la tesis
- `sections/`: Contiene los capítulos individuales
- `base/`: Configuraciones base (paquetes, estilos, colores)
- `images/`: Imágenes utilizadas en la tesis
- `bibliografia.bib`: Archivo de bibliografía en formato BibTeX

### Notas
- Los archivos generados (.aux, .log, .pdf, etc.) están ignorados en .gitignore
- Usa el estilo APA para las referencias bibliográficas
- Asegúrate de seguir los límites de páginas por capítulo según la guía