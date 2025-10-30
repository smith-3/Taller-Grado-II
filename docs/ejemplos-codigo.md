# Ejemplos de Código en TecnoTime

## Índice
1. [Introducción](#introducción)
2. [Parsing de PDFs con Regex](#parsing-de-pdfs-con-regex)
3. [Algoritmo de Backtracking](#algoritmo-de-backtracking)
4. [Estrategias de Evaluación](#estrategias-de-evaluación)
5. [Inyección de Dependencias](#inyección-de-dependencias)
6. [Queries de Base de Datos](#queries-de-base-de-datos)
7. [Manejo de Coroutines](#manejo-de-coroutines)
8. [Referencias Cruzadas](#referencias-cruzadas)

## Introducción

Este documento presenta ejemplos de código comentados de las funcionalidades más complejas de TecnoTime. Cada snippet incluye explicaciones detalladas de la lógica implementada, decisiones de diseño y consideraciones técnicas.

Los ejemplos están extraídos del código real de la aplicación y sirven como referencia para entender la implementación técnica.

## Parsing de PDFs con Regex

### Regex para Líneas de Horario

```kotlin
// Expresión regular principal para parsear líneas de horario
// Utiliza modo VERBOSE para mejor legibilidad
private val scheduleLine = Regex(
    """
(?x)                              # modo VERBOSE - ignora espacios y permite comentarios
^\s*                              # inicio de línea, espacios opcionales
(?<group>(?:[A-Z]\d+|\d+[A-Z]|\d+|[A-Z]))  # grupo: patrones como A1, 1A, 10, A
(?:\s+(?<type>\[[A-Z]{1,2}\]))?    # tipo opcional entre corchetes [P], [TP]
\s+(?<teacher>.+?)(?=\s+[A-Z]{2}\s+\d)  # docente: lazy match hasta día+hora
\s+(?<day>[A-Z]{2})                # día: exactamente 2 letras mayúsculas
\s+(?<hour>\d{1,4}\s*-\s*\d{1,4})  # hora: formato NNNN-NNNN con espacios flexibles
\s+(?<classroom>\S+)               # aula: cualquier secuencia sin espacios
\s*$                              # fin de línea, espacios opcionales
""".trimIndent(),
    RegexOption.COMMENTS           // permite comentarios en la regex
)
```

**Explicación Técnica**:
- **Grupos Nombrados**: Facilitan extracción con `match.groups["group"]?.value`
- **Lazy Matching**: `.+?` para docente evita consumir el día siguiente
- **Lookahead Positivo**: `(?=\s+[A-Z]{2}\s+\d)` asegura que el docente termine antes del día
- **Modo VERBOSE**: Espacios y comentarios hacen la regex mantenible

### Función de Parsing Principal

```kotlin
fun parse(raw: String): List<SubjectInfo> {
    Log.d(TAG, "▶️  Parse iniciando… longitud=${raw.length}")

    // 1) Fusionar líneas partidas de docente
    val merged = mergeTeacherLines(raw.lineSequence())

    val result = mutableListOf<SubjectInfo>()
    var currentHeader: SubjectInfo? = null
    var lastValidTeacher: String? = null
    var headersCount = 0
    var groupsCount = 0

    var tempBuffer = ""

    merged.forEachIndexed { idx, line ->
        val blocks = line.split(blockSeparator).filter { it.isNotBlank() }

        blocks.forEach { block ->
            val currentBlock = if (tempBuffer.isNotBlank()) "$tempBuffer $block" else block
            Log.d(TAG, "Bloque=«$currentBlock»")

            // Intentar match con cabecera de materia
            courseLine.find(currentBlock)?.let { h ->
                currentHeader = SubjectInfo(
                    level = h.groupValues[1],
                    code = h.groupValues[2],
                    name = h.groupValues[3].trim(),
                    group = null, type = null, teacher = null,
                    day = null, hour = null, classroom = null
                )
                result += currentHeader!!
                headersCount++
                lastValidTeacher = null
                tempBuffer = ""
                return@forEach
            }

            // Intentar match con línea de horario
            scheduleLine.find(currentBlock)?.let { m ->
                currentHeader?.let { base ->
                    val grp = m.groups[1]!!.value.trim()
                    val rawType = m.groups[2]?.value ?: ""
                    val type = rawType.trim('[', ']').ifBlank { "T" }
                    var teacher = m.groups[3]!!.value.trim()
                    if (teacher.startsWith("[") || teacher.isBlank()) {
                        teacher = lastValidTeacher ?: "POR DESIGNAR DOCENTE"
                    } else {
                        lastValidTeacher = teacher
                    }
                    val day = m.groups[4]!!.value
                    val hour = m.groups[5]!!.value.replace("\\s+".toRegex(), "")
                    val room = m.groups[6]!!.value

                    val info = base.copy(
                        group = grp, type = type, teacher = teacher,
                        day = day, hour = hour, classroom = room
                    )
                    result += info
                    groupsCount++
                    Log.v(TAG, "[$idx] • Grupo=$grp type=$type teacher=$teacher day=$day hour=$hour aula=$room")

                    tempBuffer = ""  // Reset buffer tras éxito
                }
            } ?: run {
                // No match: almacenar temporalmente para concatenar
                tempBuffer = currentBlock.trim()
                Log.w(TAG, "⚠️ No match scheduleLine en: «$currentBlock», almacenando para concatenar.")
            }
        }
    }

    Log.i(TAG, "✅  Parse completo: headers=$headersCount grupos=$groupsCount total=${result.size}")
    return result
}
```

**Patrones de Diseño**:
- **Buffer Temporal**: Maneja líneas que no hacen match inmediatamente
- **Estado Global**: `currentHeader` y `lastValidTeacher` mantienen contexto
- **Logging Detallado**: Facilita debugging de PDFs problemáticos

## Algoritmo de Backtracking

### Implementación del Generador

```kotlin
class ScheduleGenerator(
    private val existing: List<DayEnrolledSchedules>,
    private val strategy: ScheduleEvaluationStrategy
) {

    fun generate(
        candidates: List<List<Pair<Subject, GroupWithSchedules>>>,
        count: Int
    ): List<List<DayEnrolledSchedules>> {
        val scored = mutableListOf<Pair<List<DayEnrolledSchedules>, Double>>()

        // Función recursiva de backtracking
        fun backtrack(
            index: Int,
            chosen: List<Pair<Subject, GroupWithSchedules>>
        ) {
            if (index == candidates.size) {
                // Caso base: combinación completa
                val combined = mergeSchedules(existing, chosen)
                val score = strategy.evaluate(combined)
                scored += combined to score
                Log.d("ScheduleGen", "✅ Combination complete. Score: $score")
                return
            }

            Log.d("ScheduleGen", "🔁 Subject $index: trying ${candidates[index].size} groups")

            // Probar cada grupo para la materia actual
            for ((i, pair) in candidates[index].withIndex()) {
                val (_, gws) = pair
                Log.d(
                    "ScheduleGen",
                    "🔹 Trying group $i for subject $index → groupId=${gws.group.id}"
                )
                backtrack(index + 1, chosen + pair)
            }
        }

        backtrack(0, emptyList())

        Log.d("ScheduleGen", "🏁 Total combinations found: ${scored.size}")

        // Retornar top-N mejores (score ascendente = mejor)
        return scored
            .sortedBy { it.second }
            .take(count)
            .map { it.first }
    }
}
```

**Características Técnicas**:
- **Recursión**: Exploración depth-first del espacio de combinaciones
- **Evaluación Lazy**: Solo combina y evalúa cuando completa
- **Logging**: Tracing detallado para debugging

### Función de Fusión de Horarios

```kotlin
private fun mergeSchedules(
    base: List<DayEnrolledSchedules>,
    newGroups: List<Pair<Subject, GroupWithSchedules>>
): List<DayEnrolledSchedules> {
    // Mapeo día → lista mutable de entradas existentes
    val map = base
        .associateBy({ it.day }, { it.entries.toMutableList() })
        .toMutableMap()

    Log.d("ScheduleGen", "🔧 Merging schedules: ${newGroups.size} groups")

    // Emojis disponibles para asignación determinística
    val emojis = listOf("📘","📙","📗","📕","📓","✏️","📝","📔","📒","📚")

    // Agregar cada bloque de los grupos elegidos
    for ((subject, gws) in newGroups) {
        // Color determinístico basado en hash del código de materia
        val palette = DefaultColorPalette
        val colorOption = palette[
            subject.code.hashCode().absoluteValue % palette.size
        ]

        // Emoji determinístico
        val emoji = emojis[
            subject.code.hashCode().absoluteValue % emojis.size
        ]

        for (detail in gws.schedules) {
            Log.d(
                "ScheduleGen",
                "➕ Adding block → groupId=${gws.group.id}, day=${detail.schedule.day}, " +
                        "start=${detail.schedule.startTime}, end=${detail.schedule.endTime}"
            )

            val preview = EnrolledSchedulePreview(
                selectedSubject = SelectedSubject(
                    groupId = gws.group.id,
                    color   = colorOption.hex,
                    emoji   = emoji
                ),
                schedule    = detail.schedule,
                subject     = subject,
                group       = gws.group,
                teacher     = detail.teacher,
                classroom   = detail.classroom
            )
            // Agregar al día correspondiente
            map
                .getOrPut(detail.schedule.day) { mutableListOf() }
                .add(preview)
        }
    }

    // Reconstruir semana ordenada por día, con entradas ordenadas por hora
    return map
        .map { (day, entries) ->
            DayEnrolledSchedules(
                day     = day,
                entries = entries.sortedBy { it.schedule.startTime.toMinutes() }
            )
        }
        .sortedBy { it.day.ordinal }
}
```

**Optimizaciones**:
- **Hash Determinístico**: Colores/emojis consistentes por materia
- **Ordenamiento**: Días por ordinal, horas por minutos
- **Mutabilidad Controlada**: Map mutable solo durante construcción

## Estrategias de Evaluación

### Composite Strategy

```kotlin
class CompositeStrategy(
    private val strategies: List<ScheduleEvaluationStrategy>,
    private val weights: List<Double>
) : ScheduleEvaluationStrategy {

    init {
        require(strategies.size == weights.size) {
            "Debe haber un weight por cada strategy"
        }
    }

    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        // Suma ponderada de todas las estrategias
        return strategies.zip(weights)
            .sumOf { (strat, w) ->
                strat.evaluate(schedule) * w
            }
    }
}
```

### Minimize Gaps Strategy

```kotlin
class MinimizeGapsStrategy : ScheduleEvaluationStrategy {
    override fun evaluate(schedule: List<DayEnrolledSchedules>): Double {
        var totalGaps = 0.0

        for (dayBlock in schedule) {
            // Ordenar bloques por hora de inicio
            val sorted = dayBlock.entries
                .sortedBy { it.schedule.startTime.toMinutes() }

            // Acumular gaps entre clases consecutivas
            for (i in 0 until sorted.size - 1) {
                val endCurrent = sorted[i].schedule.endTime.toMinutes()
                val startNext  = sorted[i + 1].schedule.startTime.toMinutes()
                val gap = startNext - endCurrent
                if (gap > 0) totalGaps += gap
            }
        }

        return totalGaps
    }
}
```

**Lógica**: Calcula minutos totales de receso, menor score = mejor.

## Inyección de Dependencias

### Módulo App con Hilt

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideTecnoTimeDatabase(@ApplicationContext context: Context): TecnoTimeDatabase {
        return Room.databaseBuilder(
            context,
            TecnoTimeDatabase::class.java,
            "tecnotime.db"
        )
        .addMigrations(MIGRATION_1_2) // Manejo de migraciones
        .build()
    }

    @Provides
    @Singleton
    fun provideSubjectRepository(
        subjectDao: SubjectDao,
        mapper: SubjectMapper
    ): SubjectRepository = SubjectRepositoryImpl(subjectDao, mapper)

    @Provides
    @Singleton
    fun provideGenerateSchedulesUseCase(
        strategyFactory: ScheduleStrategyFactory,
        getAddableUc: GetAddableSubjectsHierarchyUseCase
    ): GenerateSchedulesUseCase =
        GenerateSchedulesUseCaseImpl(
            strategyFactory,
            getAddableUc
        )
}
```

**Patrones**:
- **Singleton Component**: Instancias compartidas aplicación-wide
- **Constructor Injection**: Dependencias inyectadas automáticamente
- **Abstracción**: Interfaces inyectadas, no implementaciones

## Queries de Base de Datos

### Query con Relaciones Many-to-Many

```kotlin
@Dao
interface CareerDao {

    @Query("""
        SELECT * FROM careers
        WHERE code IN (
            SELECT DISTINCT career_code
            FROM career_subject_group_cross_ref
            WHERE group_id IN (
                SELECT id FROM groups
                WHERE subject_code = :subjectCode
            )
        )
    """)
    fun findCareersBySubject(subjectCode: String): List<CareerEntity>

    @Transaction
    @Query("SELECT * FROM careers WHERE code = :code")
    fun getCareerWithLevels(code: String): CareerWithLevelsRelation

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun upsert(career: CareerEntity)
}
```

**Técnicas Avanzadas**:
- **Subqueries**: Para relaciones many-to-many
- **@Transaction**: Asegura consistencia en lecturas relacionadas
- **Upsert**: Insert o update según conflicto

### Relaciones con @Relation

```kotlin
data class CareerWithLevelsRelation(
    @Embedded val career: CareerEntity,
    @Relation(
        parentColumn = "code",
        entityColumn = "id",
        associateBy = Junction(
            value = CareerLevelCrossRef::class,
            parentColumn = "career_code",
            entityColumn = "level_id"
        )
    )
    val levels: List<LevelEntity>
)
```

**Room Relations**: Manejo automático de JOINs complejos.

## Manejo de Coroutines

### Use Case Asíncrono

```kotlin
class GenerateSchedulesUseCaseImpl @Inject constructor(
    private val strategyFactory: ScheduleStrategyFactory,
    private val getAddableUc: GetAddableSubjectsHierarchyUseCase
) : GenerateSchedulesUseCase {

    override suspend operator fun invoke(
        existing: List<DayEnrolledSchedules>,
        params: ScheduleGenerationParams
    ): List<List<DayEnrolledSchedules>> = withContext(Dispatchers.Default) {
        // Construir candidatos en IO
        val candidateGroups = buildCandidateGroups(params)

        // Crear estrategia
        val strategy: ScheduleEvaluationStrategy = strategyFactory.create(params)

        // Generar en Default (CPU-bound)
        val generator = ScheduleGenerator(existing, strategy)
        generator.generate(candidateGroups, params.numberOfSchedules)
    }

    private suspend fun buildCandidateGroups(
        params: ScheduleGenerationParams
    ): List<List<Pair<Subject, GroupWithSchedules>>> = withContext(Dispatchers.IO) {
        // Lógica de construcción...
    }
}
```

**Dispatchers**:
- **IO**: Para operaciones de BD y red
- **Default**: Para computación intensiva (backtracking)
- **Main**: Para UI (evitado en use cases)

### ViewModel con StateFlow

```kotlin
@HiltViewModel
class GenerateScheduleViewModel @Inject constructor(
    private val generateUc: GenerateSchedulesUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow(GenerateScheduleState())
    val uiState = _uiState.asStateFlow()

    fun generateSchedule() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, errorMessage = null) }
            try {
                val params = /* construir parámetros */
                val results = generateUc(existing, params)
                _uiState.update { it.copy(
                    isLoading  = false,
                    generated  = results
                ) }
                navigateToResults()
            } catch (e: Exception) {
                _uiState.update { it.copy(isLoading = false) }
                // Manejo de error...
            }
        }
    }
}
```

**Patrones Reactivos**:
- **StateFlow**: Para estado UI
- **viewModelScope**: Lifecycle-aware
- **Exception Handling**: Captura y transformación de errores

## Referencias Cruzadas

- [docs/arquitectura.md](docs/arquitectura.md): Patrones de arquitectura utilizados
- [docs/algoritmos-generacion-horarios.md](docs/algoritmos-generacion-horarios.md): Algoritmos implementados
- [docs/scraping-parsing-pdf.md](docs/scraping-parsing-pdf.md): Parsing con regex
- [docs/modelo-datos-er.md](docs/modelo-datos-er.md): Queries y relaciones
- [docs/decisiones-diseno-adr.md](docs/decisiones-diseno-adr.md): Decisiones detrás del código

Estos ejemplos ilustran la implementación técnica de TecnoTime, combinando algoritmos complejos con buenas prácticas de desarrollo Android moderno.