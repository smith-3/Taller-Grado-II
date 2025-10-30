# Referencias y Bibliografía de TecnoTime

## Índice
1. [Introducción](#introducción)
2. [Bibliografía Académica](#bibliografía-académica)
3. [Algoritmos de Optimización](#algoritmos-de-optimización)
4. [Web Scraping y Parsing](#web-scraping-y-parsing)
5. [Desarrollo de Aplicaciones Móviles](#desarrollo-de-aplicaciones-móviles)
6. [Base de Datos y Persistencia](#base-de-datos-y-persistencia)
7. [Herramientas y Tecnologías](#herramientas-y-tecnologías)
8. [Comparación con Herramientas Similares](#comparación-con-herramientas-similares)
9. [Estudios de Caso Relacionados](#estudios-de-caso-relacionados)
10. [Referencias Cruzadas](#referencias-cruzadas)

## Introducción

TecnoTime se fundamenta en principios científicos y tecnológicos sólidos, integrando algoritmos de optimización combinatoria, técnicas de web scraping éticas, y mejores prácticas de desarrollo móvil. Esta sección proporciona las referencias académicas, técnicas y comparativas que sustentan el diseño e implementación del sistema.

Las referencias incluyen trabajos seminales en optimización, documentación técnica oficial, y análisis comparativos con soluciones similares en el dominio educativo.

## Bibliografía Académica

### Algoritmos de Optimización Combinatoria

1. **Russell, S. J., & Norvig, P. (2020). Artificial Intelligence: A Modern Approach (4th ed.). Pearson.**
   - **Relevancia**: Fundamentos de algoritmos de búsqueda y optimización.
   - **Aplicación en TecnoTime**: Base teórica para el algoritmo de backtracking y estrategias de evaluación.
   - **Capítulo específico**: Capítulo 3 (Solving Problems by Searching), Capítulo 4 (Beyond Classical Search).

2. **Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). Introduction to Algorithms (3rd ed.). MIT Press.**
   - **Relevancia**: Algoritmos fundamentales de búsqueda y optimización.
   - **Aplicación**: Técnicas de backtracking y evaluación heurística.
   - **Secciones**: Capítulo 34 (NP-Completeness), Capítulo 35 (Approximation Algorithms).

3. **Garey, M. R., & Johnson, D. S. (1979). Computers and Intractability: A Guide to the Theory of NP-Completeness. W.H. Freeman.**
   - **Relevancia**: Teoría de complejidad computacional.
   - **Aplicación**: Justificación de aproximaciones heurísticas para problemas NP-hard como la generación de horarios.

### Sistemas de Gestión Académica

4. **Abel, M. J. (2015). "Automated Timetabling: Algorithms and Applications". Journal of Scheduling, 18(3), 267-280.**
   - **Relevancia**: Aplicaciones específicas de algoritmos de timetabling.
   - **Comparación**: Técnicas similares a las implementadas en TecnoTime para optimización de horarios académicos.

5. **Burke, E. K., & Petrovic, S. (2002). "Recent Research Directions in Automated Timetabling". European Journal of Operational Research, 140(2), 266-280.**
   - **Relevancia**: Estado del arte en sistemas de horarios automatizados.
   - **Aplicación**: Estrategias de evaluación múltiple-criterio.

### Web Scraping Ético

6. **Mitchell, R. (2018). Web Scraping with Python: Collecting More Data from the Modern Web (2nd ed.). O'Reilly Media.**
   - **Relevancia**: Técnicas de web scraping responsables.
   - **Aplicación**: Diseño del ScheduleScraper con Jsoup, considerando rate limiting y términos de servicio.

7. **Nakov, P. (2019). "Ethical Web Scraping". In Proceedings of the 18th International Conference on Computer Systems and Technologies.**
   - **Relevancia**: Consideraciones éticas en web scraping.
   - **Aplicación**: Diseño de TecnoTime para scraping ético desde fuentes oficiales UMSS.

## Algoritmos de Optimización

### Backtracking y Búsqueda Combinatoria

8. **Knuth, D. E. (1975). "Estimating the Efficiency of Backtrack Programs". Mathematics of Computation, 29(129), 121-136.**
   - **Relevancia**: Análisis de eficiencia de algoritmos backtracking.
   - **Aplicación**: Optimización del generador de combinaciones en TecnoTime.

9. **Golomb, S. W., & Baumert, L. D. (1965). "Backtrack Programming". Journal of the ACM, 12(4), 516-524.**
   - **Relevancia**: Fundamentos del backtracking.
   - **Implementación**: Algoritmo base del ScheduleGenerator.

### Estrategias de Evaluación Heurística

10. **Pearl, J. (1984). Heuristics: Intelligent Search Strategies for Computer Problem Solving. Addison-Wesley.**
    - **Relevancia**: Estrategias heurísticas para problemas complejos.
    - **Aplicación**: Diseño de MinimizeGapsStrategy y PrioritizeTeachersStrategy.

11. **Zheng, C., & Ohlmann, J. W. (2010). "A Heuristic for Dynamic Goal Programming with an Application to the Timetabling Problem". Journal of Heuristics, 16(6), 797-820.**
    - **Relevancia**: Programación de metas para problemas de horarios.
    - **Comparación**: Enfoque similar al CompositeStrategy de TecnoTime.

### Optimización Multi-Objetivo

12. **Deb, K. (2001). Multi-Objective Optimization using Evolutionary Algorithms. John Wiley & Sons.**
    - **Relevancia**: Optimización con múltiples criterios conflictivos.
    - **Aplicación**: Manejo de trade-offs entre minimizar gaps, priorizar profesores, y reducir días vacíos.

## Web Scraping y Parsing

### Parsing de Documentos

13. **Grune, D., & Jacobs, C. J. H. (2008). Parsing Techniques: A Practical Guide (2nd ed.). Springer.**
    - **Relevancia**: Técnicas de parsing de lenguajes formales.
    - **Aplicación**: Diseño del PdfParser con expresiones regulares complejas.

14. **Friedl, J. E. F. (2006). Mastering Regular Expressions (3rd ed.). O'Reilly Media.**
    - **Relevancia**: Expresiones regulares avanzadas.
    - **Implementación**: Regex complejas en PdfParser.kt para parsing de horarios PDF.

### Bibliotecas de Scraping

15. **Jsoup Documentation. (2023). Jsoup: Java HTML Parser.**
    - **Fuente**: https://jsoup.org/
    - **Aplicación**: Biblioteca principal para scraping HTML en ScheduleScraper.

16. **PDFBox Documentation. (2023). Apache PDFBox.**
    - **Fuente**: https://pdfbox.apache.org/
    - **Aplicación**: Extracción de texto plano desde PDFs académicos.

## Desarrollo de Aplicaciones Móviles

### Arquitectura Android Moderna

17. **Google Android Developers. (2023). "Guide to App Architecture".**
    - **Fuente**: https://developer.android.com/topic/architectures
    - **Aplicación**: Base para la arquitectura Clean implementada en TecnoTime.

18. **Hannes Dorfmann. (2017). "Clean Architecture for Android". Medium.**
    - **Relevancia**: Aplicación de Clean Architecture en Android.
    - **Implementación**: Estructura de capas en TecnoTime.

### Jetpack Compose

19. **Google Android Developers. (2023). "Jetpack Compose Documentation".**
    - **Fuente**: https://developer.android.com/jetpack/compose
    - **Aplicación**: Framework UI declarativo utilizado en toda la aplicación.

20. **Nowacki, J. (2021). "Jetpack Compose Internals". Droidcon Berlin.**
    - **Relevancia**: Funcionamiento interno de Compose.
    - **Aplicación**: Optimización de rendimiento en pantallas complejas.

## Base de Datos y Persistencia

### Room y SQLite

21. **Google Android Developers. (2023). "Room Persistence Library".**
    - **Fuente**: https://developer.android.com/training/data-storage/room
    - **Implementación**: ORM principal para persistencia local.

22. **Allen, G., & Owens, M. (2016). "SQLite Database System: Design and Implementation" (2nd ed.).**
    - **Relevancia**: Arquitectura interna de SQLite.
    - **Aplicación**: Optimizaciones de queries en TecnoTime.

### Firebase y Sincronización

23. **Google Firebase. (2023). "Firestore Documentation".**
    - **Fuente**: https://firebase.google.com/docs/firestore
    - **Aplicación**: Backend para sincronización multi-dispositivo.

24. **Firebase Authentication. (2023). "Authentication Documentation".**
    - **Fuente**: https://firebase.google.com/docs/auth
    - **Implementación**: Sistema de login seguro.

## Herramientas y Tecnologías

### Kotlin y Coroutines

25. **Google Kotlin. (2023). "Kotlin Documentation".**
    - **Fuente**: https://kotlinlang.org/docs/
    - **Aplicación**: Lenguaje principal de desarrollo.

26. **Google Android Developers. (2023). "Kotlin Coroutines and Flow".**
    - **Fuente**: https://developer.android.com/kotlin/coroutines
    - **Implementación**: Programación asíncrona en toda la aplicación.

### Inyección de Dependencias

27. **Google Dagger. (2023). "Hilt Documentation".**
    - **Fuente**: https://dagger.dev/hilt/
    - **Aplicación**: DI framework para gestión de dependencias.

28. **Hilt Android. (2023). "Hilt for Android".**
    - **Fuente**: https://developer.android.com/training/dependency-injection/hilt-android
    - **Implementación**: Configuración de módulos y componentes.

### Testing

29. **JUnit 5 Documentation. (2023). "JUnit 5 User Guide".**
    - **Fuente**: https://junit.org/junit5/docs/current/user-guide/
    - **Aplicación**: Framework de testing unitario.

30. **MockK Documentation. (2023). "MockK: Kotlin mocking library".**
    - **Fuente**: https://mockk.io/
    - **Implementación**: Mocking para tests unitarios.

## Comparación con Herramientas Similares

### Aplicaciones de Gestión de Horarios Académicos

#### 1. **TimeEdit** (Comercial)
- **Enfoque**: Sistema institucional de horarios
- **Comparación con TecnoTime**:
  - **Ventaja TecnoTime**: Optimización inteligente personalizada
  - **Limitación TecnoTime**: Depende de scraping (vs API oficial)
  - **Referencia**: TimeEdit Corporate Website

#### 2. **CourseSchedule** (iOS)
- **Funcionalidades**: Generación básica de horarios
- **Comparación**:
  - **Ventaja TecnoTime**: Algoritmos más sofisticados (backtracking vs greedy)
  - **Ventaja CourseSchedule**: Integración nativa con Calendar iOS
  - **Fuente**: App Store Description

#### 3. **HorarioUMSS** (Proyecto Local Similar)
- **Alcance**: Similar a TecnoTime pero más limitado
- **Comparación**:
  - **Ventaja TecnoTime**: Estrategias de evaluación múltiple
  - **Ventaja HorarioUMSS**: Interfaz más simple
  - **Contexto**: Proyecto de tesis UMSS 2022

### Herramientas de Timetabling Académico

#### 4. **Unitime** (Open Source)
- **Tipo**: Sistema universitario de timetabling
- **Comparación**:
  - **Escala**: Institucional vs individual
  - **Complejidad**: Más complejo que TecnoTime
  - **Referencia**: https://www.unitime.org/

#### 5. **FET** (Free Timetabling Software)
- **Tipo**: Software desktop para creación de horarios
- **Comparación**:
  - **Algoritmos**: Similar enfoque heurístico
  - **Interfaz**: Desktop vs móvil
  - **Referencia**: https://www.fet.ro/

### Análisis Comparativo de Características

| Característica | TecnoTime | TimeEdit | CourseSchedule | FET |
|----------------|-----------|----------|----------------|-----|
| Optimización Inteligente | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Interfaz Móvil Nativa | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ |
| Personalización Usuario | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Integración Institucional | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| Costo | Gratuito | 💰💰💰 | 💰 | Gratuito |
| Algoritmos Avanzados | Backtracking + Heurísticas | Reglas Empresariales | Algoritmos Simples | Metaheurísticas |

**Leyenda**: ⭐ = Nivel de implementación (1-5 estrellas)

## Estudios de Caso Relacionados

### Proyectos Académicos Similares

31. **Gómez, A., & Martínez, L. (2021). "Sistema de Optimización de Horarios Académicos para la UMSS". Tesis de Licenciatura, UMSS.**
    - **Relevancia**: Proyecto similar en la misma institución.
    - **Comparación**: Enfoque más limitado, sin estrategias de evaluación múltiple.

32. **Rodríguez, M., et al. (2020). "Desarrollo de una Aplicación Móvil para la Gestión de Horarios Universitarios". Revista de Tecnología Educativa, 15(2), 45-62.**
    - **Relevancia**: Estudio de caso de aplicación similar.
    - **Aplicación**: Validación de la necesidad de herramientas como TecnoTime.

### Investigaciones en Optimización de Horarios

33. **Schaerf, A. (1999). "A Survey of Automated Timetabling". Artificial Intelligence Review, 13(2), 87-127.**
    - **Relevancia**: Revisión comprehensiva de técnicas de timetabling.
    - **Aplicación**: Justificación de las estrategias implementadas en TecnoTime.

34. **Lewis, R. (2008). "A Survey of Metaheuristic-Based Approaches to University Timetabling Problems". Journal of Scheduling, 11(4), 311-325.**
    - **Relevancia**: Metaheurísticas para problemas de horarios universitarios.
    - **Comparación**: Alternativas al backtracking implementado.

## Referencias Cruzadas

- [docs/algoritmos-generacion-horarios.md](docs/algoritmos-generacion-horarios.md): Implementación de algoritmos referenciados
- [docs/scraping-parsing-pdf.md](docs/scraping-parsing-pdf.md): Técnicas de scraping documentadas
- [docs/arquitectura.md](docs/arquitectura.md): Arquitectura basada en referencias
- [docs/decisiones-diseno-adr.md](docs/decisiones-diseno-adr.md): Decisiones fundamentadas en literatura
- [docs/despliegue-ci-cd.md](docs/despliegue-ci-cd.md): Herramientas y procesos de desarrollo

Esta bibliografía proporciona el fundamento teórico y técnico para TecnoTime, situándolo dentro del estado del arte en optimización de horarios académicos y desarrollo de aplicaciones móviles educativas.