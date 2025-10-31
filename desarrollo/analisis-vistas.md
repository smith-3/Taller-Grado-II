# Análisis funcional de vistas y ViewModels

## Propósito
Documentar el funcionamiento de las principales vistas (Jetpack Compose) y sus contratos de ViewModel: estados, eventos, dependencias, errores, pruebas y métricas. Este análisis se basa en la implementación real del código fuente, detallando las decisiones de arquitectura tomadas y los patrones utilizados.

## Arquitectura de Presentación

### Patrón MVVM con StateFlow
Todas las vistas siguen el patrón Model-View-ViewModel con:
- **ViewModels** inyectados con Hilt, manejando estado y lógica UI
- **StateFlow** para estados reactivos que sobreviven a cambios de configuración
- **SharedFlow** para eventos únicos (navegación, errores)
- **SavedStateHandle** para persistencia de estado durante recreación de Activity

### Composables Modulares
- Composables puros con parámetros inmutables
- Preview functions para desarrollo iterativo
- Manejo de efectos secundarios con LaunchedEffect y DisposableEffect
- Tema Material 3 consistente con colores dinámicos

### Navegación Tipada
- Navigation Compose con rutas serializables
- ViewModels scoped al navigation graph
- Back stack management automático

## Índice de Vistas

### CareerSelection (Sincronización de Carreras)
**Archivos**: `presentation/careerselection/*`
**ViewModel**: `CareerSelectionViewModel`
**Propósito**: Gestionar la sincronización inicial de carreras desde UMSS

#### Estados (CareerSyncUiState)
- `LoadingCareers`: Carga inicial de carreras desde base de datos
- `CareersLoaded(careers)`: Lista de carreras disponibles
- `ConfirmEnroll(career, careers)`: Diálogo de confirmación para enrolar carrera
- `ConfirmUnenroll(career, careers)`: Diálogo de confirmación para desenrolar
- `LoadingSubjects(careers)`: Sincronización de niveles/materias/grupos
- `NavigateToLevels(career, careers)`: Navegación a vista de niveles
- `NoInternetOnCareers`: Error de red en carga inicial
- `RetryInternet`: Error de red en sincronización
- `FatalError`: Error irrecuperable

#### Eventos del ViewModel
- `loadCareers()`: Carga inicial con manejo de IOException
- `onCareerClicked(career)`: Lógica de navegación vs enrolamiento
- `syncCareer(career)`: Sincronización completa con actualización de estado
- `unenrollCareer(career)`: Remoción de carrera y cleanup de datos
- `dismissDialog()`: Reset de estado de diálogos

#### UI Dinámica
- `CareerSelectionContent`: Renderiza lista, diálogos y spinners según estado
- `CareerSelectionDialogs`: Composables especializados para confirmaciones
- `LoadingIndicatorDialog`: Spinner con mensaje contextual

#### Dependencias
- `RefreshCareersUseCase`: Obtención inicial de carreras
- `RefreshSubjectsForCareerUseCase`: Scraping de niveles/materias
- `CareerRepository`: Persistencia de estado de carreras
- `SelectedSubjectRepository`: Cleanup al desenrolar
- `UnenrollCareerUseCase`: Remoción transaccional

#### Manejo de Errores
- `IOException` → Estados de red con reintento
- `Exception` → Estado fatal sin reintento
- Estados previos preservados durante errores para UX fluida

#### Criterios de Aceptación
- Lista de carreras visible tras carga inicial
- Enrolar/desenrolar actualiza estado y navega correctamente
- Errores de red muestran diálogos apropiados con reintento
- Estado consistente tras rotación de dispositivo

#### Pruebas
- Mocks de repositorios y use cases
- Verificación de transición de estados
- Testing de manejo de excepciones
- Validación de navegación condicional

### CareerLevels y LevelDetail (Selección de Nivel y Materias)
**Archivos**: `presentation/careerselection/CareerLevelsScreen.kt`, `LevelDetailScreen.kt`
**ViewModels**: `CareerLevelsViewModel`, `LevelDetailViewModel`
**Propósito**: Exploración jerárquica de niveles y materias disponibles

#### Estados Esperados
- Carga de niveles para carrera seleccionada
- Detalle de nivel con materias y grupos
- Navegación fluida entre niveles
- Estados de carga y error consistentes

#### Eventos
- Seleccionar nivel → navegación a detalle
- Abrir detalle de materia → vista de grupos
- Seleccionar materia → navegación a AddSubjectFlow

#### Dependencias
- Datos ya sincronizados en base de datos
- Repositorios de Career, Level, Subject, Group
- Use cases de jerarquía

#### Criterios
- Listado consistente con datos Room
- Tiempos mostrados en formato configurado (12/24h)
- Navegación sin pérdida de contexto

CareerSelection
- Archivos: `presentation/careerselection/*`
- VM: `CareerSelectionViewModel`
- Estados: `CareerSyncUiState` (LoadingCareers, CareersLoaded, ConfirmEnroll, ConfirmUnenroll, LoadingSubjects, RetryInternet, FatalError, NavigateToLevels)
- Eventos: `loadCareers()`, `onCareerClicked(career)`, `syncCareer(career)`, `unenrollCareer(career)`, `dismissDialog()`
- UI dinámica: `CareerSelectionContent` renderiza lista, diálogos y spinners según estado.
- Errores: `NoInternetOnCareers`/`RetryInternet` despliegan dialogs con reintento; `FatalError` muestra acknowledgement.
- Dependencias: `RefreshCareersUseCase`, `RefreshSubjectsForCareerUseCase`, `CareerRepository`, `SelectedSubjectRepository`, `UnenrollCareerUseCase`.
- Criterios de aceptación: lista visible; enrolar/unenrolar actualiza; navegar a niveles cuando corresponde.
- Pruebas: mock de repos y use cases; verificación de transición de estados.

### AddSubjectFlow (Selección de Materia/Grupo + Personalización)
**Archivos**: `presentation/addsubject/*`
**ViewModel**: `AddSubjectViewModel`
**Propósito**: Flujo jerárquico de selección con personalización visual

#### Estados del ViewModel
- **Jerarquía**: `hierarchy` (StateFlow de CareerWithLevels)
- **Selecciones**: `selectedCareer`, `selectedLevel`, `selectedSubject`, `selectedGroup`
- **Base semanal**: `baseWeekly` para preview de conflictos
- **Editable**: `editableSelected` (SelectedSubject con color/emoji/notify)
- **Configuración**: `use24Hours` desde UserSettings

#### Flujos Derivados (combine/map)
- `careers`: Lista de carreras desde jerarquía
- `levels`: Niveles del career seleccionado
- `subjects`: Materias del level seleccionado
- `groups`: Grupos de la subject seleccionada
- `previewSchedules`: Horarios del grupo para preview

#### Eventos de Selección
- `selectCareer(c)`: Reset downstream, preserva editable
- `selectLevel(l)`: Reset subject/group, preserva editable
- `selectSubject(s)`: Reset group, preserva editable
- `selectGroup(g)`: Inicializa editableSelected con groupId

#### Personalización
- `pickColor(c)`, `pickEmoji(e)`: Actualizan editableSelected
- `toggleNotification()`, `setNotifyBeforeMinutes(m)`: Config notificaciones
- `getRandomColor()`, `getInitialRandomColor()`: Generación determinística
- `getRandomEmoji()`, `getInitialRandomEmoji()`: Desde lista predefinida

#### Persistencia de Estado
- `SavedStateHandle` para colores/emojis iniciales aleatorios
- Sobrevive a recreación de Activity sin pérdida de selección

#### UI Composables
- `AddSubjectFlowScreen`: Contenedor principal con navegación
- `SelectCareerScreen`, `SelectLevelScreen`, etc.: Pantallas especializadas
- `EmojiPickerScreen`, `ColorPickerScreen`: Pickers modales
- Preview semanal integrada con conflictos visuales

#### Manejo de Errores
- Grupos vacíos → UI muestra placeholder
- BaseWeekly faltante → fallback a lista vacía
- Validación de conflictos opcional (avoidConflicts)

#### Criterios de Aceptación
- Selección encadenada funciona correctamente
- Personalización persiste en SelectedSubject
- Preview muestra conflictos potenciales
- Navegación atrás tras confirmación exitosa

#### Pruebas
- Selección jerárquica con mocks de repos
- Persistencia de estado con SavedStateHandle
- Generación aleatoria determinística
- Estados iniciales y actualizaciones

### GenerateSchedule (Generación de Horarios)
**Archivos**: `presentation/generateSchedule/*`
**ViewModel**: `GenerateScheduleViewModel`
**Propósito**: Configuración y ejecución de algoritmo de generación

#### Estados (GenerateScheduleState)
- `isLoading`: Flag de carga
- `existing`: Horario semanal actual
- `subjects`: Lista de materias disponibles
- `selectedSubjectCodes`: Códigos específicos seleccionados
- `acceptConflicts`: Permitir solapamientos
- `minimizeGaps`: Priorizar minimizar recesos
- `prioritizeTeachers`: Favorecer docentes marcados
- `minimizeEmptyDays`: Reducir días sin clases
- `totalSubjectsCount`: Número total de materias
- `numberOfResults`: Cantidad de horarios a generar

#### Eventos (GenerateScheduleEvent)
- `ShowError(message)`: Errores de validación o ejecución
- Navegación implícita via callbacks

#### Configuración de Parámetros
- `onSubjectCodesChange(codes)`: Selección específica de materias
- `onAcceptConflictsChange(bool)`: Toggle conflictos
- `onMinimizeGapsChange(bool)`: Toggle gaps
- `onPrioritizeTeachersChange(bool)`: Toggle docentes
- `onMinimizeEmptyDaysChange(bool)`: Toggle días vacíos
- `onTotalSubjectsCountChange(count)`: Número de materias
- `onNumberOfResultsChange(count)`: Cantidad de resultados

#### Generación
- `generateSchedules(navigateToResults)`: Ejecuta algoritmo con Dispatchers.Default
- Construye `ScheduleGenerationParams` desde estado
- Llama a `generateUc` y navega a resultados

#### Aplicación de Horario
- `applySchedule(schedule, onDone)`: Persiste horario seleccionado
- Aplana entries, elimina duplicados por groupId
- Actualiza `SelectedSubject` para cada grupo
- Navega de vuelta al home

#### Dependencias
- `GetWeeklyEnrolledScheduleUseCase`: Horario actual
- `GetAddableSubjectsHierarchyUseCase`: Materias disponibles
- `GenerateSchedulesUseCase`: Algoritmo principal
- `SelectedSubjectRepository`: Persistencia

#### Criterios de Aceptación
- Configuración persiste durante sesión
- Generación produce N resultados ordenados
- Aplicación actualiza horario semanal correctamente
- Errores muestran mensajes apropiados

### Settings (Configuración General)
**Archivos**: `presentation/settings/*`
**Vistas**: `SettingsScreen`, `SettingsAddScheduleScreen`, `SettingsCareersScreen`, `SettingsTeachersScreen`
**ViewModels**: `AppSettingsViewModel`, `SettingsAddScheduleViewModel`, etc.
**Propósito**: Configuración de aplicación y preferencias de usuario

#### Funcionalidades
- **Pesos de estrategias**: Sliders para ajustar importancia de criterios de generación
- **Exportación**: PDF, Excel, imagen, JSON de horario actual
- **Importación**: JSON compartido con validación y preview
- **Formato hora**: Toggle 12/24 horas
- **Notificaciones**: Configuración global y por materia
- **Gestión de carreras**: Ver/enrolar/desenrolar carreras
- **Docentes favoritos**: Acceso a gestión de preferencias

#### Estados y Eventos
- Estados específicos por pantalla (export loading, import preview, etc.)
- Eventos de navegación y acciones (export success, import error)
- Persistencia automática en UserSettingsRepository

#### Dependencias
- `UserSettingsRepository`: Configuraciones globales
- `SelectedSubjectRepository`: Config por materia
- `GenerateWeeklySchedule*UseCase`: Exportación
- `ImportSharedScheduleUseCase`: Importación

#### Criterios de Aceptación
- Toggles y selects persisten tras reinicio
- Exportaciones generan archivos válidos
- Importaciones validan esquema y muestran preview
- Navegación fluida entre secciones

### TeacherFavorites (Gestión de Docentes Favoritos)
**Archivos**: `presentation/teachers/*`
**Vistas**: `TeacherFavoritesScreen`, `TeacherFavoritesIntroScreen`
**ViewModel**: `TeacherFavoritesViewModel`
**Propósito**: Marcar docentes que influyen en score de generación

#### Estados
- Lista de docentes disponibles con estado favorito
- Carga inicial y actualizaciones en tiempo real
- Filtros y búsqueda opcionales

#### Eventos
- Toggle favorito: Actualiza Teacher.favorite
- Persistencia automática en base de datos

#### Dependencias
- `TeacherRepository`: CRUD de docentes
- Datos poblados desde scraping de horarios

#### Criterios de Aceptación
- Favoritos persisten tras reinicio
- Reflejados en estrategias de generación (score -=1 por favorito)
- UI actualiza inmediatamente tras cambios

### Notification Management (Permisos y Demo)
**Archivos**: `presentation/notification/*` (si existe), infraestructura de notificaciones
**Vistas**: `NotificationPermissionScreen`, `NotificationDemoScreen`
**ViewModel**: `NotificationDemoViewModel`
**Propósito**: Gestión de permisos y testing de notificaciones

#### Estados
- Estado de permiso (granted/denied)
- Demo en progreso/resultados
- Configuración de notificaciones por materia

#### Eventos
- Request permission: Diálogo sistema Android 13+
- Send test notification: Emisión inmediata
- Configure per-subject: Navegación a settings

#### Dependencias
- `NotificationService`: Abstracción de sistema de notificaciones
- WorkManager para scheduling
- Permission APIs de Android

#### Criterios de Aceptación
- Permiso solicitado correctamente en Android 13+
- Notificaciones de demo aparecen en status bar
- Configuración persiste y afecta scheduling

### Welcome (Onboarding Inicial)
**Archivos**: `presentation/welcome/*`
**Vistas**: `WelcomeScreen`
**ViewModel**: `WelcomeViewModel`
**Propósito**: Primera experiencia de usuario y atajos

#### Estados
- Pantallas de onboarding secuenciales
- Atajos a sincronización y selección rápida

#### Eventos
- Skip onboarding: Navegación directa a home
- Start sync: Ir a CareerSelection
- Atajos contextuales basados en estado de app

#### Dependencias
- `UserSettingsRepository`: Marcar onboarding completado
- Estado de sincronización para atajos inteligentes

#### Criterios de Aceptación
- Flujo intuitivo para nuevos usuarios
- Atajos funcionales según contexto
- Onboarding opcional, no bloqueante

## Matriz de Funcionalidades por Vista

| Vista | Funcionalidades Principales | Estados Clave | Dependencias Críticas |
|-------|-----------------------------|---------------|----------------------|
| CareerSelection | Sincronizar carreras, enrolar/desenrolar, navegar niveles | LoadingCareers, CareersLoaded, ConfirmEnroll | RefreshCareersUseCase, CareerRepository |
| CareerLevels/LevelDetail | Explorar jerarquía, seleccionar materias | Carga niveles, detalle con grupos | Repositorios locales poblados |
| AddSubjectFlow | Selección jerárquica + personalización visual | Jerarquía cargada, selecciones, editable | GetAddableSubjectsHierarchyUseCase |
| GenerateSchedule | Configuración algoritmo, generación, aplicación | Parámetros, loading, resultados | GenerateSchedulesUseCase |
| Settings | Configuración, export/import, gestión carreras | Estados por sección, loading acciones | UserSettingsRepository, export UseCases |
| TeacherFavorites | Toggle favoritos docentes | Lista con estados | TeacherRepository |
| Notification | Permisos, demo, configuración | Granted/denied, demo results | NotificationService, WorkManager |
| Welcome | Onboarding, atajos contextuales | Secuencia onboarding | UserSettingsRepository |

## Métricas y Monitoreo

### Métricas Sugeridas por Vista
- **CareerSelection**: Tiempo carga inicial, tasa éxito sincronización, errores red
- **AddSubjectFlow**: Latencia selecciones, tiempo confirmación, abandonos flujo
- **GenerateSchedule**: Tiempo generación, calidad resultados, aplicación exitosa
- **Settings**: Éxito export/import, tiempo operaciones, crashes configuración
- **Notification**: Ratio permisos concedidos, entregas exitosas, interacciones

### Logging y Telemetría
- Logs estructurados con Timber en operaciones críticas
- Métricas de performance (tiempo generación, tamaño exports)
- Crash reporting con contexto de estado UI
- Analytics de uso (flujos completados, features utilizadas)

## Accesibilidad y UX

### Implementación de Accesibilidad
- **Labels semánticos**: `contentDescription` en todos los elementos interactivos
- **Tamaños táctiles**: Mínimo 48dp según Material Guidelines
- **Contraste**: Colores que cumplen ratio 4.5:1 mínimo
- **Navegación**: Focus traversal lógico, keyboard navigation
- **Screen readers**: Labels descriptivos, estado anunciado

### Navegación por Teclado
- Tab order natural siguiendo flujo visual
- Enter/Espacio para activar elementos
- Escape para cerrar modales
- Shortcuts contextuales (Ctrl+S para guardar, etc.)

## Manejo de Errores y Estados de Error

### Estrategias de Error Handling
- **Red**: Diálogos de reintento con exponential backoff, estado offline preservado
- **Datos**: Placeholders con mensajes guía, estados de carga parcial
- **Validación**: Mensajes inline, prevención de estados inválidos
- **Recuperación**: Estados previos restaurables, navegación segura

### Estados de Error por Vista
- **Loading states**: Skeletons/Shimmers para mejor perceived performance
- **Empty states**: Ilustraciones y CTAs para guiar usuario
- **Error states**: Mensajes accionables con opciones de retry
- **Offline states**: Funcionalidad degradada con indicadores claros

## Recomendaciones de Mejora

### Performance
- **Lazy loading**: Implementar pagination en listas largas
- **Background prefetch**: Cargar datos anticipadamente
- **Memory optimization**: Liberar recursos en onStop, usar WeakReferences

### UX Enhancements
- **Progress indicators**: Barras de progreso en operaciones largas
- **Undo/Redo**: Para acciones destructivas (desenrolar, aplicar horario)
- **Deep linking**: Compartir estados específicos de la app
- **Haptic feedback**: Confirmaciones táctiles para acciones importantes

### Arquitectura
- **Feature modules**: Separar vistas en módulos Gradle independientes
- **Compose testing**: Aumentar cobertura de UI tests
- **State preservation**: Mejorar handling de process death
- **A11y testing**: Automatizar validaciones de accesibilidad

### Analytics y Métricas
- **User journeys**: Tracking completo de flujos críticos
- **Performance monitoring**: APM integrado (Firebase Performance)
- **Crash analysis**: Context-rich crash reports
- **A/B testing**: Framework para experimentos de UX
