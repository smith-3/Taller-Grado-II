# Plan de Estructura Técnica Detallada: "TecnoTime"

Este documento vincula la estructura académica de la tesis con la implementación real del código, especificando las librerías, clases y tecnologías que sustentan cada sección.

## Elementos Preliminares
- **Carátula, Dedicatoria, Agradecimientos, Índices, Ficha Resumen.**

---

## Capítulo I: Introducción
*Visión general del proyecto.*
1.1. Antecedentes
1.2. Planteamiento del Problema
1.3. Objetivos
1.4. Justificación
1.5. Alcances y Límites

---

## Capítulo II: Marco Teórico (Fundamentación Técnica)
*Aquí se definen los conceptos y se menciona qué tecnología específica los implementa en la App.*

**2.1. Ingeniería de Software para Dispositivos Móviles**
*   2.1.1. **Arquitectura Android & Kotlin:** Ventajas del desarrollo nativo moderno.
    *   *Sustento en Código:* Uso de **Kotlin** (Coroutines, Flow) y **Android SDK 35**.
*   2.1.2. **Interfaz Declarativa (Jetpack Compose):** Cambio de paradigma imperativo a declarativo.
    *   *Sustento en Código:* Librerías `androidx.compose.material3`, `androidx.compose.ui`. Componentes como `LazyColumn`, `Scaffold`.
*   2.1.3. **Gestión de Estado Reactiva (UDF):** Flujo unidireccional de datos.
    *   *Sustento en Código:* Uso de `StateFlow` y `collectAsStateWithLifecycle` en la UI.

**2.2. Arquitectura de Software**
*   2.2.1. **Clean Architecture:** Separación de responsabilidades.
    *   *Sustento en Código:* Paquetes `domain` (UseCases, Models), `data` (Repositories, Sources), `presentation` (ViewModels, Screens).
*   2.2.2. **Patrón MVVM:** Desacoplamiento de lógica y vista.
    *   *Sustento en Código:* Clases `ViewModel` de Jetpack, `HiltViewModel`.
*   2.2.3. **Inyección de Dependencias (DI):** Inversión de control.
    *   *Sustento en Código:* Librería **Hilt (Dagger)**. Anotaciones `@Inject`, `@Module`, `@Provides`.

**2.3. Algoritmia de Planificación (Timetabling)**
*   2.3.1. **Problemas de Satisfacción de Restricciones (CSP):** Modelado matemático.
    *   *Sustento en Código:* Clases `Subject`, `Group`, `Schedule`. Restricciones de "no solape" y "cupos".
*   2.3.2. **Algoritmos de Búsqueda (Backtracking):** Exploración del espacio de soluciones.
    *   *Sustento en Código:* Lógica recursiva en `ScheduleGenerator.kt`.
*   2.3.3. **Heurísticas de Optimización:** Poda y ordenamiento.
    *   *Sustento en Código:* Estrategias `MinimizeGapsStrategy`, `PrioritizeTeachersStrategy`.

**2.4. Inteligencia Artificial en el Borde (Edge AI)**
*   2.4.1. **Inferencia Local (On-Device):** Privacidad y funcionamiento offline.
    *   *Sustento en Código:* Ejecución local sin llamadas a API externas (OpenAI/Gemini).
*   2.4.2. **Modelos de Lenguaje (LLMs) y Cuantización:** Optimización de recursos.
    *   *Sustento en Código:* Uso de **llama.cpp** (librería C++ portada a Android) para ejecutar modelos cuantizados (formato `.gguf`, ej. Q4_K_M). Clases `LlamaAndroidClient`, `ModelDownloader`.

**2.5. Gestión y Persistencia de Datos**
*   2.5.1. **Persistencia Relacional Local:** Bases de datos embebidas.
    *   *Sustento en Código:* Librería **Room** (abstracción de SQLite). Entidades `@Entity`, DAOs `@Dao`.
*   2.5.2. **Estrategias Offline-First:** Sincronización diferida.
    *   *Sustento en Código:* Librería **WorkManager** para tareas en segundo plano (`SyncWorker`).

---

## Capítulo III: Área de Aplicación
*Contexto UMSS.*
3.1. Descripción del entorno (UMSS, FCyT).
3.2. Proceso actual de inscripción.
3.3. Identificación de problemas (Choques, falta de planificación).
3.4. Requerimientos (Funcionales: Generar, Exportar, IA; No Funcionales: Offline, Rendimiento).

---

## Capítulo IV: Metodología y Desarrollo (Implementación)
*Descripción detallada de cómo se construyó cada módulo del código.*

**4.1. Metodología Adoptada**
*   Enfoque iterativo, uso de Git/GitHub.

**4.2. F1: Ingestión y Normalización de Datos**
*   *Objetivo:* Obtener datos de horarios.
*   *Implementación:* `PdfExtractor` (lectura de PDF crudo), `PdfParser` (Regex para detectar materias/grupos), `ScheduleImportRepository`.

**4.3. F2: Modelo de Dominio y Persistencia**
*   *Objetivo:* Estructurar la información.
*   *Implementación:* Definición de entidades en Room (`SubjectEntity`, `GroupEntity`). Relaciones 1:N.

**4.4. F3: Motor de Generación de Horarios**
*   *Objetivo:* Crear combinaciones válidas.
*   *Implementación:* `GenerateSchedulesUseCase`. Algoritmo de Backtracking con poda por conflictos. Puntuación de horarios (Scoring).

**4.5. F4: Arquitectura Offline-First y Sincronización**
*   *Objetivo:* Funcionar sin internet.
*   *Implementación:* `NetworkConnectivityChecker`. Lógica de "Single Source of Truth" en Repositorios. `AutoSyncUseCase`.

**4.6. F5: Interoperabilidad (JSON)**
*   *Objetivo:* Compartir horarios.
*   *Implementación:* Serialización JSON (`Kotlin Serialization` o `Gson`). Esquema de intercambio definido en `ShareableScheduleDto`.

**4.7. F6: Experiencia de Usuario (UX) y Navegación**
*   *Objetivo:* Flujos de usuario fluidos.
*   *Implementación:* `Navigation Compose` (NavHost). Pantallas `Onboarding`, `Home`, `ScheduleView`. Widget de escritorio (`Glance` o `RemoteViews`).

**4.8. F7: Asistente Inteligente (Simón)**
*   *Objetivo:* Ayuda contextual con IA.
*   *Implementación:* Integración de **llama.cpp** vía JNI. Gestión de descarga de modelos (`DownloadManager`). Prompt Engineering en `AiPromptManager`.

**4.9. F8: Exportación de Resultados**
*   *Objetivo:* Sacar el horario de la app.
*   *Implementación:* Generación de gráficos (Canvas/Bitmap) para Imagen. Generación de PDF (librería PDF nativa o iText). CSV/Excel.

**4.10. F9: Pruebas y Calidad**
*   *Objetivo:* Asegurar estabilidad.
*   *Implementación:* Unit Tests (`JUnit`, `Mockk`). UI Tests (`Compose Test Rule`).

**4.11. F10: Seguridad y Privacidad**
*   *Objetivo:* Proteger datos.
*   *Implementación:* Permisos de Android (Storage, Notifications). Datos 100% locales.

**4.12. F11: Empaquetado y Release**
*   *Objetivo:* App lista para producción.
*   *Implementación:* Configuración de `build.gradle` (ProGuard/R8 rules), optimización de recursos.

---

## Capítulo V: Conclusiones y Recomendaciones
5.1. Conclusiones.
5.2. Recomendaciones.

## Bibliografía y Anexos
