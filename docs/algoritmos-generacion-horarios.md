# Algoritmos de Generación de Horarios en TecnoTime

## Índice
1. [Introducción](#introducción)
2. [Arquitectura del Sistema de Generación](#arquitectura-del-sistema-de-generación)
3. [Algoritmo de Backtracking para Combinaciones](#algoritmo-de-backtracking-para-combinaciones)
4. [Estrategias de Evaluación](#estrategias-de-evaluación)
5. [Pseudocódigo Detallado](#pseudocódigo-detallado)
6. [Flujo de Generación Completo](#flujo-de-generación-completo)
7. [Optimizaciones y Consideraciones](#optimizaciones-y-consideraciones)
8. [Referencias Cruzadas](#referencias-cruzadas)

## Introducción

La generación inteligente de horarios constituye el corazón funcional de TecnoTime, transformando datos académicos crudos en horarios optimizados que satisfacen múltiples criterios de usuario. Este proceso combina algoritmos de búsqueda combinatoria con estrategias de evaluación heurística para producir resultados prácticos en tiempo razonable.

El sistema implementa un enfoque híbrido que combina backtracking para exploración exhaustiva con evaluación estratégica para priorización de soluciones óptimas.

## Arquitectura del Sistema de Generación

### Componentes Principales

1. **ScheduleGenerator**: Clase principal que implementa el algoritmo de backtracking
2. **ScheduleEvaluationStrategy**: Interfaz para estrategias de evaluación
3. **ScheduleStrategyFactory**: Fábrica que crea estrategias compuestas según preferencias
4. **GenerateSchedulesUseCase**: Caso de uso que coordina el proceso completo

### Patrón Strategy para Evaluación

El sistema utiliza el patrón Strategy para permitir múltiples criterios de evaluación:

```kotlin
fun interface ScheduleEvaluationStrategy {
    fun evaluate(schedule: List<DayEnrolledSchedules>): Double
}
```

Esto permite composibilidad: diferentes estrategias pueden combinarse con pesos variables.

## Algoritmo de Backtracking para Combinaciones

### Principio Básico

El backtracking explora sistemáticamente todas las combinaciones posibles de grupos de materias, descartando ramas que violan restricciones (como conflictos de horario) y evaluando las completas con estrategias heurísticas.

### Implementación en TecnoTime

```kotlin
fun generate(
    candidates: List<List<Pair<Subject, GroupWithSchedules>>>,
    count: Int
): List<List<DayEnrolledSchedules>> {
    val scored = mutableListOf<Pair<List<DayEnrolledSchedules>, Double>>()

    fun backtrack(
        index: Int,
        chosen: List<Pair<Subject, GroupWithSchedules>>
    ) {
        if (index == candidates.size) {
            val combined = mergeSchedules(existing, chosen)
            val score = strategy.evaluate(combined)
            scored += combined to score
            return
        }

        for ((i, pair) in candidates[index].withIndex()) {
            backtrack(index + 1, chosen + pair)
        }
    }

    backtrack(0, emptyList())

    return scored
        .sortedBy { it.second }
        .take(count)
        .map { it.first }
}
```

### Complejidad Computacional

- **Espacio de Búsqueda**: Para n materias con g grupos cada una, el espacio es O(g^n)
- **Poda**: Conflictos de horario reducen significativamente las ramas exploradas
- **Evaluación**: Solo las combinaciones completas se evalúan, limitando el costo

## Estrategias de Evaluación

### Estrategia Compuesta

La estrategia principal combina múltiples criterios con pesos:

```kotlin
class CompositeStrategy(
    private val strategies: List<ScheduleEvaluationStrategy>,
    private val weights: List<Double>
) : ScheduleEvaluationStrategy {

    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        return strategies.zip(weights)
            .sumOf { (strat, w) ->
                strat.evaluate(schedule) * w
            }
    }
}
```

### Estrategias Individuales

#### 1. MinimizeGapsStrategy
```kotlin
class MinimizeGapsStrategy : ScheduleEvaluationStrategy {
    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        var totalGaps = 0.0

        for (dayBlock in schedule) {
            val sorted = dayBlock.entries
                .sortedBy { it.schedule.startTime.toMinutes() }

            for (i in 0 until sorted.size - 1) {
                val endCurrent = sorted[i].schedule.endTime.toMinutes()
                val startNext = sorted[i + 1].schedule.startTime.toMinutes()
                val gap = startNext - endCurrent
                if (gap > 0) totalGaps += gap
            }
        }

        return totalGaps
    }
}
```

**Lógica**: Calcula minutos totales de receso entre clases consecutivas. Menor score = mejor (menos gaps).

#### 2. AcceptConflictsStrategy
```kotlin
class AcceptConflictsStrategy(
    private val acceptConflicts: Boolean
) : ScheduleEvaluationStrategy {
    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        if (acceptConflicts) return 0.0 // No penalizar

        var conflicts = 0
        for (dayBlock in schedule) {
            val entries = dayBlock.entries
            for (i in entries.indices) {
                for (j in i + 1 until entries.size) {
                    if (timesOverlap(entries[i], entries[j])) {
                        conflicts++
                    }
                }
            }
        }
        return conflicts * 1000.0 // Penalización alta
    }
}
```

**Lógica**: Detecta superposiciones temporales. Penaliza fuertemente si no se permiten conflictos.

#### 3. PrioritizeTeachersStrategy
```kotlin
class PrioritizeTeachersStrategy : ScheduleEvaluationStrategy {
    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        var nonFavoritePenalty = 0.0
        for (dayBlock in schedule) {
            for (entry in dayBlock.entries) {
                if (!entry.teacher.favorite) {
                    nonFavoritePenalty += 1.0
                }
            }
        }
        return nonFavoritePenalty
    }
}
```

**Lógica**: Penaliza horarios con docentes no favoritos marcados.

#### 4. MinimizeEmptyDaysStrategy
```kotlin
class MinimizeEmptyDaysStrategy : ScheduleEvaluationStrategy {
    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        val emptyDays = schedule.count { it.entries.isEmpty() }
        return emptyDays.toDouble()
    }
}
```

**Lógica**: Cuenta días sin clases. Menos días vacíos = mejor distribución.

### Configuración Dinámica

La fábrica crea estrategias basadas en parámetros de usuario:

```kotlin
fun create(params: ScheduleGenerationParams): ScheduleEvaluationStrategy {
    val list = mutableListOf<ScheduleEvaluationStrategy>()
    val weights = mutableListOf<Double>()

    // Siempre incluir conflictos
    list += AcceptConflictsStrategy(params.acceptConflicts)
    weights += 1.0

    // Opcionales según preferencias
    if (params.minimizeGaps) {
        list += MinimizeGapsStrategy()
        weights += 1.0
    }

    if (params.prioritizeFavoriteTeachers) {
        list += PrioritizeTeachersStrategy()
        weights += 1.0
    }

    // Siempre incluir días vacíos
    list += MinimizeEmptyDaysStrategy()
    weights += 0.5

    return CompositeStrategy(list, weights)
}
```

## Pseudocódigo Detallado

### Algoritmo Principal de Generación

```
fun generarHorarios(candidatos, numeroResultados, estrategia):
    resultados = lista vacía de (horario, puntuacion)

    funcion backtrack(indice, seleccionados):
        si indice == candidatos.longitud:
            horario_combinado = fusionarHorarios(horario_existente, seleccionados)
            puntuacion = estrategia.evaluar(horario_combinado)
            resultados.agregar((horario_combinado, puntuacion))
            retornar

        para cada opcion en candidatos[indice]:
            backtrack(indice + 1, seleccionados + opcion)

    backtrack(0, lista vacía)

    retornar resultados
        .ordenar_por_puntuacion_ascendente()
        .tomar_primeros(numeroResultados)
        .mapear_a_horarios()

```

### Función de Fusión de Horarios

```
fun fusionarHorarios(base, nuevosGrupos):
    mapa = base.agrupar_por_dia()

    para cada (materia, grupoConHorarios) en nuevosGrupos:
        color = generar_color_deterministico(materia.codigo)
        emoji = seleccionar_emoji_deterministico(materia.codigo)

        para cada horarioDetalle en grupoConHorarios.horarios:
            entrada = EnrolledSchedulePreview(
                materia: materia,
                grupo: grupoConHorarios.grupo,
                horario: horarioDetalle.horario,
                docente: horarioDetalle.docente,
                aula: horarioDetalle.aula,
                color: color,
                emoji: emoji
            )
            mapa[horarioDetalle.dia].agregar(entrada)

    retornar mapa
        .mapear_a_lista_diaria()
        .ordenar_por_dia()
        .ordenar_entradas_por_hora()
```

## Flujo de Generación Completo

```mermaid
graph TD
    A[Usuario configura parámetros] --> B[GenerateSchedulesUseCase.invoke]
    B --> C[buildCandidateGroups]
    C --> D[Obtener jerarquía de materias]
    D --> E[Filtrar por códigos específicos o cantidad]
    E --> F[Crear pares Subject-GroupWithSchedules]
    F --> G[Crear estrategia con ScheduleStrategyFactory]
    G --> H[ScheduleGenerator.generate]
    H --> I[Inicializar backtrack con índice 0]
    I --> J{Índice == candidatos.size?}
    J -->|Sí| K[fusionarHorarios con existentes]
    J -->|No| L[Para cada grupo candidato]
    L --> M[backtrack(indice+1, seleccionados + grupo)]
    M --> J
    K --> N[strategy.evaluate]
    N --> O[Agregar a resultados puntuados]
    O --> P[Backtrack retorna]
    P --> Q[Ordenar por puntuación ascendente]
    Q --> R[Tomar top-N resultados]
    R --> S[Retornar List<List<DayEnrolledSchedules>>]
    S --> T[Presentar opciones al usuario]
```

## Optimizaciones y Consideraciones

### Optimizaciones Implementadas

1. **Poda Temprana**: Conflictos se detectan durante backtracking, no solo en evaluación
2. **Evaluación Lazy**: Solo combinaciones completas se evalúan
3. **Ordenamiento Determinístico**: Colores y emojis basados en hash para consistencia
4. **Paralelización**: Uso de Dispatchers.Default para computación intensiva

### Limitaciones

- **Explosión Combinatoria**: Con muchas materias, el espacio crece exponencialmente
- **Evaluación Heurística**: Las estrategias son aproximaciones, no garantizan óptimo global
- **Dependencia de Datos**: Calidad de resultados depende de precisión del scraping

### Mejoras Potenciales

- **Algoritmos Genéticos**: Para espacios de búsqueda muy grandes
- **Machine Learning**: Aprender preferencias de usuario
- **Cache de Resultados**: Memorizar evaluaciones previas
- **Poda Avanzada**: Estimaciones de cota inferior para poda temprana

## Referencias Cruzadas

- [docs/arquitectura.md](docs/arquitectura.md): Arquitectura general y patrones utilizados
- [docs/funcionalidades.md](docs/funcionalidades.md): Funcionalidades de generación de horarios
- [docs/decisiones-diseno-adr.md](docs/decisiones-diseno-adr.md): Decisiones de diseño para algoritmos
- [docs/ejemplos-codigo.md](docs/ejemplos-codigo.md): Implementación detallada del backtracking
- [docs/modelo-datos-er.md](docs/modelo-datos-er.md): Estructura de datos de horarios

Los algoritmos de generación de horarios en TecnoTime demuestran cómo combinar técnicas clásicas de búsqueda con estrategias modernas de evaluación para resolver problemas complejos de optimización combinatoria en el dominio académico.