# Decisiones de Diseño Arquitectural (ADRs) en TecnoTime

## Índice
1. [Introducción al Formato ADR](#introducción-al-formato-adr)
2. [ADR-001: Adopción de Clean Architecture](#adr-001-adopción-de-clean-architecture)
3. [ADR-002: Elección de Room sobre SQLDelight](#adr-002-elección-de-room-sobre-sqldelight)
4. [ADR-003: Patrón Strategy para Evaluación de Horarios](#adr-003-patrón-strategy-para-evaluación-de-horarios)
5. [ADR-004: Inyección de Dependencias con Hilt](#adr-004-inyección-de-dependencias-con-hilt)
6. [ADR-005: Backtracking vs Algoritmos Genéticos](#adr-005-backtracking-vs-algoritmos-genéticos)
7. [ADR-006: Regex Complejas para Parsing de PDFs](#adr-006-regex-complejas-para-parsing-de-pdfs)
8. [ADR-007: Modelo de Datos Many-to-Many](#adr-007-modelo-de-datos-many-to-many)
9. [Referencias Cruzadas](#referencias-cruzadas)

## Introducción al Formato ADR

Las Architecture Decision Records (ADRs) documentan decisiones arquitecturalmente significativas tomadas durante el desarrollo de TecnoTime. Cada ADR sigue el formato estándar:

- **Contexto**: Situación que motivó la decisión
- **Opciones Consideradas**: Alternativas evaluadas
- **Decisión**: Opción elegida y justificación
- **Consecuencias**: Impactos positivos y negativos
- **Estado**: Implementado, Propuesto, etc.

## ADR-001: Adopción de Clean Architecture

### Contexto
TecnoTime requiere una arquitectura que soporte:
- Mantenibilidad a largo plazo
- Testabilidad independiente de frameworks
- Evolución tecnológica sin afectar lógica de negocio
- Separación clara de responsabilidades

### Opciones Consideradas
1. **Clean Architecture**: Capas independientes con inversión de dependencias
2. **MVVM Tradicional**: Sin separación de dominio y datos
3. **Hexagonal Architecture**: Similar a Clean pero más compleja
4. **Onion Architecture**: Variante de Clean Architecture

### Decisión
Adoptar Clean Architecture con las siguientes capas:
- **Presentación**: ViewModels, UI, navegación
- **Dominio**: Casos de uso, modelos, servicios
- **Datos**: Repositorios, entidades, APIs externas

### Justificación
- **Independencia Tecnológica**: La lógica de negocio no depende de Android
- **Testabilidad**: Cada capa se puede testear aisladamente
- **Mantenibilidad**: Cambios en UI no afectan algoritmos
- **Escalabilidad**: Nuevas funcionalidades siguen el mismo patrón

### Consecuencias
**Positivas**:
- Código altamente testeable
- Fácil migración tecnológica (ej: cambiar Room por SQLDelight)
- Lógica de negocio reutilizable

**Negativas**:
- Mayor complejidad inicial
- Más clases y interfaces
- Curva de aprendizaje para nuevos desarrolladores

**Estado**: Implementado

## ADR-002: Elección de Room sobre SQLDelight

### Contexto
Necesidad de una capa de persistencia robusta para datos académicos complejos con relaciones many-to-many.

### Opciones Consideradas
1. **Room**: ORM oficial de Android con Kotlin integration
2. **SQLDelight**: Multiplataforma, compile-time verification
3. **Realm**: Base de datos orientada a objetos
4. **SQLite Directo**: Sin abstracción ORM

### Decisión
Utilizar Room como ORM principal.

### Justificación
- **Integración Nativa**: Parte del Android Jetpack, mantenido por Google
- **Kotlin First**: Soporte nativo para data classes y coroutines
- **Relaciones Complejas**: Excelente manejo de many-to-many con Junction tables
- **Migraciones**: Sistema robusto de migraciones de esquema
- **Observabilidad**: LiveData/Flow integration

### Consecuencias
**Positivas**:
- Menos boilerplate que SQLite directo
- Type safety en queries
- Integración perfecta con Architecture Components

**Negativas**:
- Vendor lock-in con Android
- No multiplataforma (vs SQLDelight)
- Mayor tamaño de APK

**Estado**: Implementado

## ADR-003: Patrón Strategy para Evaluación de Horarios

### Contexto
El algoritmo de generación de horarios necesita múltiples criterios de evaluación configurables por el usuario.

### Opciones Consideradas
1. **Strategy Pattern**: Interfaces funcionales con implementaciones específicas
2. **Chain of Responsibility**: Cadena de evaluadores
3. **Composite Pattern**: Árbol de evaluadores
4. **Hardcoded Conditions**: If-else anidados

### Decisión
Implementar patrón Strategy con CompositeStrategy para combinar criterios.

### Justificación
- **Flexibilidad**: Nuevos criterios sin modificar código existente
- **Configurabilidad**: Usuario puede activar/desactivar criterios
- **Composability**: Combinar estrategias con pesos diferentes
- **Testabilidad**: Cada estrategia se puede testear independientemente

### Consecuencias
**Positivas**:
- Fácil agregar nuevos criterios (ej: minimizar traslados)
- Configuración dinámica por usuario
- Código limpio y extensible

**Negativas**:
- Mayor número de clases
- Complejidad en weighting de estrategias

**Estado**: Implementado

## ADR-004: Inyección de Dependencias con Hilt

### Contexto
Gestión de dependencias complejas en aplicación con múltiples capas y repositorios.

### Opciones Consideradas
1. **Hilt**: DI oficial de Android basado en Dagger
2. **Koin**: Ligero, Kotlin-first
3. **Manual DI**: Constructor injection sin framework
4. **Service Locator**: Singleton global

### Decisión
Adoptar Hilt para inyección de dependencias.

### Justificación
- **Android Official**: Recomendado por Google, integración perfecta
- **Compile-time Safety**: Errores de DI detectados en compilación
- **Performance**: Optimizado para Android
- **Escalabilidad**: Maneja complejidad de grandes aplicaciones

### Consecuencias
**Positivas**:
- Menos boilerplate que DI manual
- Detección temprana de errores
- Integración con ViewModels y WorkManager

**Negativas**:
- Curva de aprendizaje pronunciada
- Mayor tiempo de compilación
- Debugging más complejo

**Estado**: Implementado

## ADR-005: Backtracking vs Algoritmos Genéticos

### Contexto
Necesidad de algoritmo eficiente para generar combinaciones óptimas de horarios académicos.

### Opciones Consideradas
1. **Backtracking**: Exploración sistemática con poda
2. **Algoritmos Genéticos**: Evolución poblacional
3. **Programación Dinámica**: Optimización matemática
4. **Búsqueda Local**: Hill climbing

### Decisión
Implementar backtracking con evaluación heurística.

### Justificación
- **Garantía de Óptimo**: Explora todas combinaciones válidas
- **Determinismo**: Resultados consistentes
- **Simplicidad**: Algoritmo bien entendido
- **Eficiencia**: Poda temprana reduce espacio de búsqueda

### Consecuencias
**Positivas**:
- Resultados predecibles y óptimos
- Fácil de entender y debuggear
- Buen rendimiento para problemas medianos

**Negativas**:
- Escalabilidad limitada para problemas grandes
- No paralelizable fácilmente
- Requiere poda inteligente

**Estado**: Implementado

## ADR-006: Regex Complejas para Parsing de PDFs

### Contexto
Parsing de PDFs académicos con formato variable requiere expresiones regulares robustas.

### Opciones Consideradas
1. **Regex Complejas**: Patrones avanzados con grupos nombrados
2. **Parser Personalizado**: Lógica imperativa de parsing
3. **Biblioteca de Parsing**: PDF parsing libraries especializadas
4. **Machine Learning**: Modelos para reconocimiento de patrones

### Decisión
Utilizar expresiones regulares complejas con modo VERBOSE y grupos nombrados.

### Justificación
- **Mantenibilidad**: Regex documentadas son más legibles
- **Performance**: Más rápido que parsing personalizado
- **Flexibilidad**: Manejo de variaciones de formato
- **Debugging**: Grupos nombrados facilitan troubleshooting

### Consecuencias
**Positivas**:
- Código conciso para lógica compleja
- Fácil modificación de patrones
- Buen rendimiento

**Negativas**:
- Difícil de leer para principiantes
- Frágil ante cambios de formato
- Debugging requiere expertise en regex

**Estado**: Implementado

## ADR-007: Modelo de Datos Many-to-Many

### Contexto
Relaciones complejas entre carreras, niveles, materias y grupos requieren modelo many-to-many.

### Opciones Consideradas
1. **Junction Tables**: Tablas intermedias con foreign keys
2. **JSON Embedding**: Almacenar arrays en campos JSON
3. **Separate Entities**: Entidades intermedias con lógica
4. **NoSQL**: Documentos anidados

### Decisión
Implementar junction tables con Room relations.

### Justificación
- **Normalización**: Evita duplicación de datos
- **Integridad**: Constraints referenciales
- **Queries**: SQL eficiente para relaciones complejas
- **Room Support**: Relaciones @Relation nativas

### Consecuencias
**Positivas**:
- Modelo relacional correcto
- Queries eficientes
- Integridad de datos

**Negativas**:
- Queries más complejas
- Mayor número de tablas
- Migraciones más delicadas

**Estado**: Implementado

## Referencias Cruzadas

- [docs/arquitectura.md](docs/arquitectura.md): Implementación de la arquitectura elegida
- [docs/modelo-datos-er.md](docs/modelo-datos-er.md): Modelo de datos resultante
- [docs/algoritmos-generacion-horarios.md](docs/algoritmos-generacion-horarios.md): Algoritmos implementados
- [docs/scraping-parsing-pdf.md](docs/scraping-parsing-pdf.md): Técnicas de parsing
- [docs/ejemplos-codigo.md](docs/ejemplos-codigo.md): Implementación práctica de patrones

Las ADRs de TecnoTime documentan decisiones que priorizan mantenibilidad, testabilidad y escalabilidad sobre simplicidad inmediata, estableciendo una base sólida para evolución futura.