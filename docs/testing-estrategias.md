# Estrategias de Testing en TecnoTime

## Índice
1. [Introducción](#introducción)
2. [Pirámide de Testing](#pirámide-de-testing)
3. [Testing Unitario](#testing-unitario)
4. [Testing de Integración](#testing-de-integración)
5. [Testing de UI](#testing-de-ui)
6. [Testing de Algoritmos](#testing-de-algoritmos)
7. [Mocking y Test Doubles](#mocking-y-test-doubles)
8. [Cobertura de Código](#cobertura-de-código)
9. [CI/CD con Testing](#cicd-con-testing)
10. [Referencias Cruzadas](#referencias-cruzadas)

## Introducción

TecnoTime implementa una estrategia de testing comprehensiva que abarca desde pruebas unitarias de algoritmos complejos hasta tests de integración end-to-end. La arquitectura Clean permite testing aislado de cada capa, mientras que las estrategias de mocking facilitan pruebas determinísticas.

Este documento detalla las estrategias de testing utilizadas, con ejemplos prácticos y consideraciones para mantener calidad de código.

## Pirámide de Testing

```
     ┌─────────────┐  (Menos tests, más caros)
     │  E2E Tests  │  ~5-10 tests
     │  UI Tests   │
     └─────────────┘
           │
     ┌─────────────┐
     │Integration │  ~50-100 tests
     │   Tests     │
     └─────────────┘
           │
     ┌─────────────┐
     │  Unit Tests │  ~500+ tests  (Más tests, más baratos)
     └─────────────┘
```

**Distribución en TecnoTime**:
- **Unit Tests**: 70% - Algoritmos, utilidades, mappers
- **Integration Tests**: 20% - Repositorios, use cases
- **UI Tests**: 10% - Flujos críticos de usuario

## Testing Unitario

### Configuración con JUnit 5 y MockK

```kotlin
// build.gradle.kts
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.9.2")
    testImplementation("io.mockk:mockk:1.13.4")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.1")
    testImplementation("androidx.arch.core:core-testing:2.2.0")
}
```

### Ejemplo: Testing de Strategy

```kotlin
class MinimizeGapsStrategyTest {

    private lateinit var strategy: MinimizeGapsStrategy

    @BeforeEach
    fun setup() {
        strategy = MinimizeGapsStrategy()
    }

    @Test
    fun `evaluate returns zero for schedule with no gaps`() {
        // Given: horario sin gaps
        val schedule = listOf(
            DayEnrolledSchedules(
                day = WeekDay.MONDAY,
                entries = listOf(
                    createScheduleEntry("08:00", "10:00"),
                    createScheduleEntry("10:00", "12:00")  // sin gap
                )
            )
        )

        // When
        val score = strategy.evaluate(schedule)

        // Then
        assertEquals(0.0, score)
    }

    @Test
    fun `evaluate calculates total gaps correctly`() {
        // Given: horario con gaps
        val schedule = listOf(
            DayEnrolledSchedules(
                day = WeekDay.MONDAY,
                entries = listOf(
                    createScheduleEntry("08:00", "09:00"),  // 60 min gap
                    createScheduleEntry("10:00", "11:00"),  // 30 min gap
                    createScheduleEntry("11:30", "12:00")
                )
            )
        )

        // When
        val score = strategy.evaluate(schedule)

        // Then: 60 + 30 = 90 minutos de gap
        assertEquals(90.0, score)
    }

    private fun createScheduleEntry(start: String, end: String): EnrolledSchedulePreview {
        return mockk<EnrolledSchedulePreview>().apply {
            every { schedule.startTime } returns TimeOfDay.fromString(start)
            every { schedule.endTime } returns TimeOfDay.fromString(end)
        }
    }
}
```

**Patrones de Testing**:
- **Given-When-Then**: Estructura clara de tests
- **Mocking**: Aislar unidad bajo test
- **Nombres Descriptivos**: Tests como documentación

### Testing de Funciones Puras

```kotlin
class ScheduleStrategyFactoryTest {

    private lateinit var factory: ScheduleStrategyFactory

    @BeforeEach
    fun setup() {
        factory = ScheduleStrategyFactory()
    }

    @Test
    fun `create with minimize gaps returns composite with correct strategies`() {
        // Given
        val params = ScheduleGenerationParams(
            minimizeGaps = true,
            prioritizeFavoriteTeachers = false,
            acceptConflicts = true
        )

        // When
        val strategy = factory.create(params)

        // Then
        assertTrue(strategy is CompositeStrategy)
        val composite = strategy as CompositeStrategy

        // Verificar estrategias incluidas
        val strategies = composite.strategies
        assertTrue(strategies.any { it is AcceptConflictsStrategy })
        assertTrue(strategies.any { it is MinimizeGapsStrategy })
        assertTrue(strategies.any { it is MinimizeEmptyDaysStrategy })
        assertFalse(strategies.any { it is PrioritizeTeachersStrategy })
    }
}
```

## Testing de Integración

### Testing de Repositorios

```kotlin
class SubjectRepositoryImplTest {

    private lateinit var repository: SubjectRepositoryImpl
    private lateinit var dao: SubjectDao
    private lateinit var mapper: SubjectMapper

    @BeforeEach
    fun setup() {
        dao = mockk()
        mapper = mockk()
        repository = SubjectRepositoryImpl(dao, mapper)
    }

    @Test
    fun `getSubjectsByCareer returns mapped subjects`(): Unit = runBlocking {
        // Given
        val careerCode = "409701"
        val entities = listOf(
            SubjectEntity("2006063", "FISICA GENERAL", false, false, true)
        )
        val domain = listOf(
            Subject("2006063", "FISICA GENERAL", false, false, true)
        )

        coEvery { dao.getSubjectsByCareer(careerCode) } returns entities
        every { mapper.entityToDomain(entities[0]) } returns domain[0]

        // When
        val result = repository.getSubjectsByCareer(careerCode)

        // Then
        assertEquals(domain, result)
        coVerify { dao.getSubjectsByCareer(careerCode) }
        verify { mapper.entityToDomain(entities[0]) }
    }
}
```

**Características**:
- **Coroutines Testing**: `runBlocking` para tests suspend
- **Mock Verification**: Verificar llamadas a dependencias
- **Data Flow**: Testear transformación entidad → dominio

### Testing de Use Cases

```kotlin
class GenerateSchedulesUseCaseImplTest {

    private lateinit var useCase: GenerateSchedulesUseCaseImpl
    private lateinit var strategyFactory: ScheduleStrategyFactory
    private lateinit var getAddableUc: GetAddableSubjectsHierarchyUseCase

    @BeforeEach
    fun setup() {
        strategyFactory = mockk()
        getAddableUc = mockk()
        useCase = GenerateSchedulesUseCaseImpl(strategyFactory, getAddableUc)
    }

    @Test
    fun `invoke generates schedules with correct parameters`(): Unit = runBlocking {
        // Given
        val existing = emptyList<DayEnrolledSchedules>()
        val params = ScheduleGenerationParams(
            totalSubjectsCount = 3,
            numberOfSchedules = 5
        )

        val mockHierarchy = mockk<List<CareerWithLevels>>()
        val mockStrategy = mockk<ScheduleEvaluationStrategy>()

        coEvery { getAddableUc() } returns mockHierarchy
        every { strategyFactory.create(params) } returns mockStrategy

        // Mock del generador para verificar parámetros
        val mockGenerator = mockk<ScheduleGenerator>()
        val expectedResult = listOf(mockk<List<DayEnrolledSchedules>>())

        // When - necesitamos mockear la construcción interna
        // Esto requiere diseño testable (inyectar generador o usar spy)

        // Then
        // Verificar que se llamaron los colaboradores correctos
        coVerify { getAddableUc() }
        verify { strategyFactory.create(params) }
    }
}
```

## Testing de UI

### Configuración con Compose Testing

```kotlin
// build.gradle.kts
androidTestImplementation("androidx.compose.ui:ui-test-junit4:1.4.3")
androidTestImplementation("io.mockk:mockk-android:1.13.4")
```

### Ejemplo: Testing de Pantalla de Generación

```kotlin
class GenerateScheduleScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    private lateinit var viewModel: GenerateScheduleViewModel
    private lateinit var mockGenerateUc: GenerateSchedulesUseCase

    @Before
    fun setup() {
        mockGenerateUc = mockk()
        viewModel = GenerateScheduleViewModel(mockGenerateUc)
    }

    @Test
    fun `generate button triggers schedule generation`() {
        // Given
        val expectedParams = ScheduleGenerationParams(
            totalSubjectsCount = 5,
            numberOfSchedules = 3
        )

        coEvery { mockGenerateUc(any(), expectedParams) } returns emptyList()

        composeTestRule.setContent {
            GenerateScheduleConfigScreen(viewModel = viewModel)
        }

        // When: click generate button
        composeTestRule
            .onNodeWithText("Generar Horarios")
            .performClick()

        // Then: verify use case was called
        coVerify { mockGenerateUc(any(), expectedParams) }
    }

    @Test
    fun `loading indicator shows during generation`() {
        // Given: viewModel in loading state
        viewModel.setLoadingState(true)

        composeTestRule.setContent {
            GenerateScheduleConfigScreen(viewModel = viewModel)
        }

        // Then: loading indicator is displayed
        composeTestRule
            .onNodeWithContentDescription("Cargando")
            .assertIsDisplayed()
    }
}
```

**Técnicas de UI Testing**:
- **Semantic Matching**: Buscar por texto, descripción
- **Actions**: Clicks, scrolls, inputs
- **Assertions**: Visibility, text content, hierarchy

## Testing de Algoritmos

### Testing del Backtracking

```kotlin
class ScheduleGeneratorTest {

    private lateinit var generator: ScheduleGenerator
    private lateinit var mockStrategy: ScheduleEvaluationStrategy

    @BeforeEach
    fun setup() {
        mockStrategy = mockk()
        generator = ScheduleGenerator(emptyList(), mockStrategy)
    }

    @Test
    fun `generate explores all combinations for simple case`() {
        // Given: 2 materias, 2 grupos cada una = 4 combinaciones
        val candidates = listOf(
            listOf(
                Subject("MAT1", "Materia 1") to mockGroupWithSchedules(1),
                Subject("MAT1", "Materia 1") to mockGroupWithSchedules(2)
            ),
            listOf(
                Subject("MAT2", "Materia 2") to mockGroupWithSchedules(3),
                Subject("MAT2", "Materia 2") to mockGroupWithSchedules(4)
            )
        )

        every { mockStrategy.evaluate(any()) } returns 0.0

        // When
        val result = generator.generate(candidates, 10)

        // Then: debería generar 4 combinaciones
        assertEquals(4, result.size)
        verify(exactly = 4) { mockStrategy.evaluate(any()) }
    }

    @Test
    fun `generate returns only requested count when more combinations exist`() {
        // Given: muchas combinaciones pero solo queremos top 2
        val candidates = createLargeCandidateSet()
        every { mockStrategy.evaluate(any()) } returnsMany listOf(1.0, 2.0, 0.5, 3.0) // scores

        // When
        val result = generator.generate(candidates, 2)

        // Then: solo 2 resultados, los mejores scores
        assertEquals(2, result.size)
        // Verificar ordenamiento por score ascendente
    }

    private fun mockGroupWithSchedules(id: Long): GroupWithSchedules {
        return mockk<GroupWithSchedules>().apply {
            every { group.id } returns id
            every { schedules } returns emptyList()
        }
    }
}
```

**Desafíos del Testing de Algoritmos**:
- **Determinismo**: Asegurar resultados predecibles
- **Complejidad**: Tests para casos edge
- **Performance**: Tests que no sean demasiado lentos

### Testing de Regex Parsing

```kotlin
class PdfParserTest {

    private lateinit var parser: PdfParser

    @BeforeEach
    fun setup() {
        parser = PdfParser()
    }

    @Test
    fun `parse handles simple schedule line correctly`() {
        // Given: línea de horario simple
        val input = "A 2006063 FISICA GENERAL\n1A [TP] JUAN PEREZ LU 0800-1000 A-101"

        // When
        val result = parser.parse(input)

        // Then
        assertEquals(2, result.size) // header + schedule

        val header = result[0]
        assertEquals("A", header.level)
        assertEquals("2006063", header.code)
        assertEquals("FISICA GENERAL", header.name)

        val schedule = result[1]
        assertEquals("1A", schedule.group)
        assertEquals("TP", schedule.type)
        assertEquals("JUAN PEREZ", schedule.teacher)
        assertEquals("LU", schedule.day)
        assertEquals("0800-1000", schedule.hour)
        assertEquals("A-101", schedule.classroom)
    }

    @ParameterizedTest
    @ValueSource(strings = [
        "1A [TP] JUAN PEREZ LU 0800-1000 A-101",
        "B2 INGLES MA 1400-1600 B-201",
        "10 [P] MARIA GONZALEZ MI 0900-1100 C-301"
    ])
    fun `scheduleLine regex matches various formats`(line: String) {
        // Given: línea de horario
        val regex = parser.scheduleLine

        // When
        val match = regex.find(line)

        // Then
        assertNotNull(match)
        assertNotNull(match.groups["group"])
        assertNotNull(match.groups["teacher"])
        assertNotNull(match.groups["day"])
        assertNotNull(match.groups["hour"])
        assertNotNull(match.groups["classroom"])
    }
}
```

## Mocking y Test Doubles

### Uso de MockK

```kotlin
class FirebaseAuthManagerTest {

    private lateinit var authManager: FirebaseAuthManager
    private lateinit var mockFirebaseAuth: FirebaseAuth
    private lateinit var mockUser: FirebaseUser

    @BeforeEach
    fun setup() {
        mockFirebaseAuth = mockk()
        mockUser = mockk()
        authManager = FirebaseAuthManager(mockFirebaseAuth)
    }

    @Test
    fun `login success returns user data`(): Unit = runBlocking {
        // Given
        val email = "test@umss.edu.bo"
        val password = "password123"

        every { mockFirebaseAuth.signInWithEmailAndPassword(email, password) } returns
            mockk<Task<AuthResult>>().apply {
                every { isSuccessful } returns true
                every { result.user } returns mockUser
                every { addOnCompleteListener(any()) } answers {
                    val listener = arg<OnCompleteListener<AuthResult>>(0)
                    listener.onComplete(this@apply)
                }
            }

        every { mockUser.uid } returns "user123"
        every { mockUser.email } returns email

        // When
        val result = authManager.login(email, password)

        // Then
        assertTrue(result.isSuccess)
        assertEquals("user123", result.getOrNull()?.id)
    }
}
```

**Tipos de Test Doubles**:
- **Mocks**: Verificar interacciones
- **Stubs**: Proporcionar datos predefinidos
- **Spies**: Wrap de objetos reales con verificación

## Cobertura de Código

### Configuración de Jacoco

```kotlin
// build.gradle.kts
android {
    buildTypes {
        getByName("debug") {
            isTestCoverageEnabled = true
        }
    }
}

tasks.register("jacocoTestReport", JacocoReport::class) {
    dependsOn("testDebugUnitTest", "createDebugCoverageReport")

    reports {
        xml.required.set(true)
        html.required.set(true)
        csv.required.set(false)
    }

    classDirectories.setFrom(
        fileTree(dir: "${buildDir}/tmp/kotlin-classes/debug") {
            exclude("**/R.class", "**/R$*.class", "**/BuildConfig.class")
        }
    )

    sourceDirectories.setFrom(files("${project.projectDir}/src/main/java"))
    executionData.setFrom(fileTree(dir: buildDir, includes: listOf(
        "jacoco/testDebugUnitTest.exec",
        "outputs/code_coverage/debugAndroidTest/connected/**/*.ec"
    )))
}
```

### Métricas Objetivo

- **Unit Tests**: >80% cobertura de líneas
- **Integration Tests**: >70% cobertura de branches
- **Critical Paths**: 100% cobertura en algoritmos de generación

### Análisis de Cobertura

```bash
# Generar reporte
./gradlew jacocoTestReport

# Ver reporte HTML
open app/build/reports/jacoco/jacocoTestReport/html/index.html
```

## CI/CD con Testing

### GitHub Actions Workflow

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: ~/.gradle/caches
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

    - name: Run unit tests
      run: ./gradlew testDebugUnitTest

    - name: Run instrumentation tests
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: 29
        script: ./gradlew connectedDebugAndroidTest

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: app/build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml
```

### Quality Gates

- **Tests Pass**: Todos los tests deben pasar
- **Coverage Threshold**: Mínimo 75% cobertura total
- **Mutation Testing**: Opcional con PIT para calidad de tests
- **Static Analysis**: Detekt para code quality

## Referencias Cruzadas

- [docs/arquitectura.md](docs/arquitectura.md): Arquitectura testable
- [docs/algoritmos-generacion-horarios.md](docs/algoritmos-generacion-horarios.md): Algoritmos testeados
- [docs/decisiones-diseno-adr.md](docs/decisiones-diseno-adr.md): Decisiones que afectan testing
- [docs/despliegue-ci-cd.md](docs/despliegue-ci-cd.md): CI/CD con testing integrado
- [docs/ejemplos-codigo.md](docs/ejemplos-codigo.md): Código bajo test

La estrategia de testing de TecnoTime asegura calidad y mantenibilidad, con énfasis en algoritmos críticos y flujos de usuario principales.