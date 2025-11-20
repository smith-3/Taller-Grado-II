# Plan Detallado de Contenido: Capítulo 2 - Marco Teórico
## "Aplicación Móvil para la Gestión de Horarios Universitarios"

Este documento desglosa el contenido técnico específico que se redactará en el Marco Teórico, asegurando que cada concepto académico esté respaldado por la implementación real del proyecto "TecnoTime" y referencias bibliográficas formales.

---

### 2.1. Ingeniería de Software para Dispositivos Móviles
*Objetivo: Fundamentar la elección de tecnologías modernas de desarrollo nativo.*

#### 2.1.1. Arquitectura de Plataforma Android
*   **Concepto Teórico:** Estructura del SO (Kernel Linux, ART, Framework de Aplicaciones). Ventajas del desarrollo nativo (rendimiento, acceso a hardware) vs. híbrido.
*   **Implementación en Código:**
    *   Uso de **Android SDK 35** (Target SDK) y **Kotlin** como lenguaje de primera clase.
    *   Uso de **Coroutines** y **Flow** para manejo asíncrono eficiente.
*   **Referencias:** Documentación oficial de Android Developers; *Android Programming: The Big Nerd Ranch Guide*.

#### 2.1.2. Paradigma de Interfaz Declarativa (Jetpack Compose)
*   **Concepto Teórico:** Diferencia entre UI Imperativa (manipulación del DOM/Views) y UI Declarativa (UI como función del estado). Concepto de "Recomposición" inteligente.
*   **Implementación en Código:**
    *   Librería: **Jetpack Compose** (`androidx.compose.ui`, `androidx.compose.material3`).
    *   Componentes clave: `Scaffold`, `LazyColumn`, `Surface`.
*   **Referencias:**
    *   Singh, R. et al. (2025). "Comparative study of XML vs Jetpack Compose".
    *   Jain, K., & Patel, N. (2023). "Jetpack Compose vs. XML: A Comparative Analysis".

#### 2.1.3. Ciclo de Vida y Gestión de Estado (UDF)
*   **Concepto Teórico:** Patrón **Unidirectional Data Flow (UDF)**. El estado baja, los eventos suben. Inmutabilidad del estado.
*   **Implementación en Código:**
    *   Librería: **Kotlin StateFlow** (`MutableStateFlow`, `asStateFlow`).
    *   Clases: `AiManagementViewModel` expone un `UiState` inmutable.
*   **Referencias:** Documentación de Android Architecture Components; *Reactive Programming with Kotlin*.

---

### 2.2. Arquitectura de Software
*Objetivo: Justificar la estructura del proyecto y los patrones de diseño para mantenibilidad.*

#### 2.2.1. Clean Architecture (Arquitectura Limpia)
*   **Concepto Teórico:** Principio de separación de responsabilidades y Regla de Dependencia (capas internas no conocen a las externas).
*   **Implementación en Código:**
    *   Paquetes explícitos: `domain`, `data`, `presentation`.
*   **Referencias:**
    *   Martin, R. C. (2017). *Clean Architecture: A Craftsman's Guide to Software Structure and Design*.

#### 2.2.2. Patrón Model-View-ViewModel (MVVM)
*   **Concepto Teórico:** Separación de la lógica de presentación de la interfaz gráfica.
*   **Implementación en Código:**
    *   Clases: `androidx.lifecycle.ViewModel`.
*   **Referencias:** Patrones de diseño de Microsoft/Google; *Pro Android with Kotlin*.

#### 2.2.3. Inyección de Dependencias (DI)
*   **Concepto Teórico:** Inversión de Control (IoC) para desacoplar clases y facilitar testing.
*   **Implementación en Código:**
    *   Librería: **Hilt** (basada en Dagger).
*   **Referencias:** Documentación de Dagger/Hilt; *Dependency Injection Principles, Practices, and Patterns*.

---

### 2.3. Algoritmia de Planificación (Timetabling)
*Objetivo: Explicar la "magia" matemática detrás del generador de horarios.*

#### 2.3.1. Problemas de Satisfacción de Restricciones (CSP)
*   **Concepto Teórico:** Definición formal de CSP (Variables, Dominios, Restricciones).
*   **Implementación en Código:**
    *   Variables: Materias seleccionadas. Dominios: Grupos disponibles.
*   **Referencias:**
    *   Russell, S. J., & Norvig, P. (2021). *Artificial Intelligence: A Modern Approach* (4th ed.). (Capítulo sobre CSP).

#### 2.3.2. Algoritmos de Búsqueda: Backtracking (Vuelta Atrás)
*   **Concepto Teórico:** Exploración sistemática del árbol de soluciones.
*   **Implementación en Código:**
    *   Clase: `ScheduleGenerator.kt`. Método: `backtrack()`.
*   **Referencias:** Russell & Norvig (2021); *Introduction to Algorithms* (Cormen et al.).

#### 2.3.3. Heurísticas de Poda y Optimización
*   **Concepto Teórico:** Técnicas para reducir el espacio de búsqueda (Pruning, Fail-First).
*   **Implementación en Código:**
    *   Poda por restricciones duras; Scoring por preferencias.
*   **Referencias:** Literatura académica sobre Timetabling (ej. *Practice and Theory of Automated Timetabling*).

---

### 2.4. Inteligencia Artificial en el Borde (Edge AI)
*Objetivo: Detallar la innovación tecnológica del asistente "Simón".*

#### 2.4.1. Inferencia Local (On-Device) vs. Cloud
*   **Concepto Teórico:** Ejecución de modelos de IA en el hardware del usuario. Privacidad, Latencia, Offline.
*   **Implementación en Código:**
    *   Tecnología: **llama.cpp** vía JNI.
*   **Referencias:**
    *   Pothineni, S. H. (2024). "Offline-First Mobile Architecture".
    *   Papers recientes sobre Edge AI (ej. *Edge Artificial Intelligence*, arXiv).

#### 2.4.2. Modelos de Lenguaje (LLMs) y Cuantización
*   **Concepto Teórico:** Reducción de precisión de pesos para viabilizar LLMs en móviles.
*   **Implementación en Código:**
    *   Modelo: Cuantizado `.gguf` (Q4_K_M).
*   **Referencias:** Papers sobre Quantization-aware training y GGUF format.

---

### 2.5. Gestión y Persistencia de Datos
*Objetivo: Explicar cómo la app guarda datos y funciona sin internet.*

#### 2.5.1. Persistencia Relacional Local (ORM)
*   **Concepto Teórico:** Bases de datos relacionales embebidas (SQLite).
*   **Implementación en Código:**
    *   Librería: **Room**.
*   **Referencias:** Documentación de SQLite; *Database System Concepts*.

#### 2.5.2. Estrategias Offline-First y Sincronización
*   **Concepto Teórico:** "Single Source of Truth" (SSOT). Sincronización diferida.
*   **Implementación en Código:**
    *   Librería: **WorkManager**.
*   **Referencias:**
    *   Pothineni, S. H. (2024). "Offline-First Mobile Architecture".
    *   Guda, V. R. (2025). "Implementation of Offline-First Architectures".
