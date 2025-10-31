
# Capítulo IV · Metodología y Desarrollo

Este directorio documenta el proceso seguido para construir TecnoTime como si fuera desarrollado de cero por un único desarrollador, sin teoría general de metodologías. Se describe la metodología adoptada, sus fases y el trabajo realizado en cada etapa, junto con requisitos, historias de usuario, backlog, trazabilidad y criterios de aceptación.

## Metodología Adoptada

### Enfoque Pragmático e Iterativo
Se adoptó un enfoque de desarrollo pragmático e iterativo, inspirado en prácticas ágiles pero adaptado a un contexto de desarrollo individual. La metodología se centró en:

- **Iteraciones cortas**: Cada etapa representa una iteración funcional completa
- **Prototipado rápido**: Implementación directa sin excesiva planificación previa
- **Validación continua**: Cada etapa produce un entregable funcional y testeable
- **Refinamiento incremental**: Mejoras y ajustes basados en el aprendizaje de cada iteración

### Principios Fundamentales
1. **Funcionalidad sobre perfección**: Priorizar features que funcionen sobre código perfecto
2. **Arquitectura emergente**: Dejar que la arquitectura evolucione naturalmente con los requerimientos
3. **Testing práctico**: Enfocarse en tests que validen comportamientos críticos
4. **Documentación viva**: Mantener documentación actualizada con el código

## Arquitectura Técnica Implementada

### Patrón Clean Architecture
TecnoTime implementa Clean Architecture con tres capas principales:

#### Capa de Presentación (Presentation)
- **Framework**: Jetpack Compose para UI declarativa
- **Navegación**: Navigation Compose con rutas tipadas
- **Estado**: ViewModels con StateFlow y SharedFlow
- **Inyección**: Hilt para dependencias en ViewModels

**Decisiones clave tomadas**:
- ViewModels manejan estado UI y eventos de navegación
- StateFlow para estados reactivos, SharedFlow para eventos únicos
- Composables modulares con preview functions para desarrollo rápido

#### Capa de Dominio (Domain)
- **Casos de uso**: Lógica de negocio pura, independiente de frameworks
- **Repositorios**: Interfaces que definen contratos de datos
- **Modelos**: Entidades de dominio inmutables
- **Servicios**: Estrategias y algoritmos de negocio

**Decisiones clave tomadas**:
- Casos de uso como funciones suspend que encapsulan operaciones complejas
- Interfaces de repositorio para abstracción de datos
- Patrón Strategy para algoritmos intercambiables (generación de horarios)

#### Capa de Datos (Data)
- **ORM**: Room con DAOs transaccionales
- **Scraping**: Jsoup para parsing HTML/PDF
- **Sincronización**: Firebase para backup remoto
- **Mappers**: Conversión entre entidades de dominio y base de datos

**Decisiones clave tomadas**:
- Transacciones Room para integridad de datos
- Upsert operations para sincronización idempotente
- Parsing con regex configurables para adaptarse a cambios en fuentes externas

### Herramientas y Tecnologías Utilizadas

#### Desarrollo y Build
- **Lenguaje**: Kotlin 1.9+ con características modernas (coroutines, flows, sealed classes)
- **IDE**: Android Studio con plugins de Compose
- **Build System**: Gradle Kotlin DSL
- **Version Control**: Git con GitHub para CI/CD

#### Frameworks y Librerías
- **UI**: Jetpack Compose 1.5+ con Material 3
- **Arquitectura**: Hilt para DI, Navigation Compose
- **Base de Datos**: Room 2.6+ con migraciones
- **Trabajo en Background**: WorkManager para notificaciones y sync
- **Red**: Retrofit para APIs, Jsoup para scraping
- **Fechas**: ThreeTenABP para compatibilidad pre-API 26
- **Imágenes**: Coil para carga asíncrona
- **JSON**: Gson para serialización

#### Testing
- **Unit Tests**: JUnit 5, MockK para mocking
- **Integration Tests**: Room in-memory, Hilt testing
- **UI Tests**: Compose testing, Espresso para fragments legacy

### Desafíos Enfrentados y Soluciones

#### Desafío: Complejidad del Algoritmo de Generación
**Problema**: Generar horarios óptimos requiere explorar un espacio combinatorio grande (backtracking).
**Solución**: Implementación de backtracking con poda temprana y estrategias de evaluación ponderadas.

#### Desafío: Sincronización de Datos Externos
**Problema**: Los PDFs de UMSS cambian formato frecuentemente.
**Solución**: Regex configurables y validación robusta con fallbacks.

#### Desafío: Estado UI Complejo
**Problema**: Flujos de selección jerárquica (carrera→nivel→materia→grupo) con preview.
**Solución**: StateFlow combinados y SavedStateHandle para persistencia.

#### Desafío: Notificaciones en Background
**Problema**: Scheduling preciso de notificaciones con WorkManager.
**Solución**: Workers con constraints de red y batería, cálculo de delays.

### Resultados Obtenidos

#### Métricas de Rendimiento
- **Tiempo de generación**: < 2-3 segundos para 6×3 combinaciones típicas
- **Tamaño APK**: ~15MB optimizado con R8
- **Consumo batería**: Workers agrupados, ejecución eficiente
- **Compatibilidad**: Android 7.0+ (API 24) con 95% cobertura de dispositivos

#### Calidad de Código
- **Cobertura de tests**: 70%+ en lógica crítica
- **Arquitectura**: Separación clara de responsabilidades
- **Mantenibilidad**: Código modular y bien documentado
- **Escalabilidad**: Fácil añadir nuevas estrategias y features

#### Experiencia de Usuario
- **Onboarding intuitivo**: Flujo de sincronización guiado
- **Generación rápida**: Resultados en segundos con opciones múltiples
- **Personalización**: Colores, emojis, docentes favoritos
- **Persistencia**: Datos offline con sync automática

## Fases de Desarrollo

- 4.1 Metodología y fases: 4.1-metodologia-y-fases.md
- 4.2 Etapa 1 (Planificación y requisitos): 4.2-etapa-1-planificacion-requisitos.md
- 4.3 Etapa 2 (Datos y scraping): 4.3-etapa-2-datos-y-scraping.md
- 4.4 Etapa 3 (Arquitectura y setup): 4.4-etapa-3-arquitectura-setup.md
- 4.5 Etapa 4 (Dominio y generación de horarios): 4.5-etapa-4-dominio-generacion-horarios.md
- 4.6 Etapa 5 (UI/UX y flujos): 4.6-etapa-5-ui-ux-flujos.md
- 4.7 Etapa 6 (Notificaciones y sincronización): 4.7-etapa-6-notificaciones-sincronizacion.md
- 4.8 Etapa 7 (Pruebas y calidad): 4.8-etapa-7-pruebas-calidad.md
- 4.9 Etapa 8 (Entrega y despliegue): 4.9-etapa-8-entrega-despliegue.md

## Soportes Complementarios

- Requisitos (funcionales y no funcionales): requisitos.md
- Historias de usuario y criterios de aceptación: historias-de-usuario.md
- Backlog y tablero (texto): backlog-tareas.md
- Trazabilidad (HU → módulos/archivos): trazabilidad.md
- Definición de Terminado (DoD): criterios-aceptacion-definicion-de-terminado.md
- Análisis funcional de vistas: analisis-vistas.md

## Documentos Relacionados

- Arquitectura: docs/arquitectura.md
- Funcionalidades: docs/funcionalidades.md
- Flujos de usuario: docs/flujos-usuario.md
- Scraping y parsing: docs/scraping-parsing-pdf.md
- Modelo de datos ER: docs/modelo-datos-er.md
- Notificaciones: docs/notificaciones.md
- Sincronización automática: docs/sincronizacion-automatica.md
- Testing: docs/testing-estrategias.md
- CI/CD y despliegue: docs/despliegue-ci-cd.md