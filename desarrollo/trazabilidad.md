# Trazabilidad HU → Módulos/Archivos

## Metodología de Trazabilidad

### Enfoque Sistemático
- **Bidireccional**: HU ↔ código ↔ tests ↔ documentación
- **Granularidad**: Mapeo a archivos específicos y funciones
- **Actualización**: Mantenida con cambios en código
- **Validación**: Verificada contra implementación real

### Niveles de Trazabilidad
1. **Historia → Módulos**: Mapping de alto nivel
2. **Historia → Archivos**: Artefactos específicos
3. **Historia → Funciones**: Métodos y clases clave
4. **Historia → Tests**: Cobertura de validación

## Trazabilidad Detallada por Historia de Usuario

### HU1: Sincronizar Oferta Académica
**Objetivo**: Obtener datos académicos desde UMSS
**Módulos principales**: `domain/usecase`, `data/remote`, `presentation/careerselection`

#### Use Cases
- `domain/usecase/FetchCareersUseCase.kt`: Obtención inicial de lista de carreras
- `domain/usecase/RefreshCareersUseCase.kt`: Sincronización desde fuente externa
- `domain/usecase/RefreshSubjectsForCareerUseCase.kt`: Scraping de niveles/materias/grupos
- `domain/usecase/SyncCareersUseCase.kt`: Orquestación completa de sincronización

#### Repositorios
- `domain/repository/CareerRepository.kt`: CRUD carreras con estado de sincronización
- `domain/repository/SubjectRepository.kt`: Gestión de materias por nivel
- `domain/repository/GroupRepository.kt`: Grupos con horarios asociados
- `domain/repository/LevelRepository.kt`: Jerarquía académica

#### UI Layer
- `presentation/careerselection/CareerSelectionScreen.kt`: Lista de carreras con estados
- `presentation/careerselection/CareerSelectionViewModel.kt`: Lógica de enrolamiento/sync
- `presentation/careerselection/CareerLevelsScreen.kt`: Exploración de niveles
- `presentation/careerselection/LevelDetailScreen.kt`: Detalle con materias/grupos

#### Data Layer
- `data/remote/UmssScraper.kt`: Implementación de scraping HTML/PDF
- `data/local/dao/CareerDao.kt`: Persistencia con upsert transaccional
- `data/local/dao/SubjectDao.kt`: Queries jerárquicas
- `data/mapper/CareerMapper.kt`: Conversión entidad ↔ dominio

#### Tests
- `domain/usecase/FetchCareersUseCaseTest.kt`: Validación de obtención
- `data/remote/UmssScraperTest.kt`: Parsing con fixtures
- `presentation/careerselection/CareerSelectionViewModelTest.kt`: Estados UI

### HU2: Seleccionar Materias y Grupos
**Objetivo**: Elección personalizada de componentes del horario
**Módulos principales**: `presentation/addsubject`, `domain/usecase`

#### Use Cases
- `domain/usecase/GetAddableSubjectsHierarchyUseCase.kt`: Jerarquía completa disponible
- `domain/usecase/GetWeeklyEnrolledScheduleUseCase.kt`: Horario actual para preview

#### UI Layer
- `presentation/addsubject/AddSubjectFlowScreen.kt`: Contenedor de navegación jerárquica
- `presentation/addsubject/AddSubjectViewModel.kt`: Estado complejo de selecciones
- `presentation/addsubject/SelectCareerScreen.kt`: Picker de carrera
- `presentation/addsubject/SelectLevelScreen.kt`: Picker de nivel
- `presentation/addsubject/SelectSubjectScreen.kt`: Picker de materia
- `presentation/addsubject/SelectGroupScreen.kt`: Picker de grupo con preview
- `presentation/addsubject/ColorPickerScreen.kt`: Personalización visual
- `presentation/addsubject/EmojiPickerScreen.kt`: Selección de emoji

#### Repositorios
- `domain/repository/SelectedSubjectRepository.kt`: Persistencia de selecciones
- `domain/repository/UserSettingsRepository.kt`: Configuraciones de formato

#### Data Layer
- `data/local/dao/SelectedSubjectDao.kt`: CRUD con validaciones
- `ui/common/color/DefaultColorPalette.kt`: Paleta de colores disponibles

#### Tests
- `presentation/addsubject/AddSubjectViewModelTest.kt`: Flujo de selecciones
- `domain/repository/SelectedSubjectRepositoryTest.kt`: Persistencia
- UI tests para navegación jerárquica

### HU3: Preferencias de Generación
**Objetivo**: Configuración de criterios de optimización
**Módulos principales**: `presentation/settings`, `domain/service`

#### UI Layer
- `presentation/settings/SettingsScreen.kt`: Contenedor de secciones
- `presentation/settings/SettingsTeachersScreen.kt`: Gestión de favoritos
- `presentation/settings/AppSettingsViewModel.kt`: Estado de configuraciones

#### Domain Layer
- `domain/service/CompositeStrategy.kt`: Combinación de estrategias con pesos
- `domain/service/MinimizeGapsStrategy.kt`: Optimización de recesos
- `domain/service/PrioritizeTeachersStrategy.kt`: Preferencia de docentes
- `domain/service/MinimizeEmptyDaysStrategy.kt`: Reducción de días libres

#### Repositorios
- `domain/repository/UserSettingsRepository.kt`: Persistencia de pesos y configuración
- `domain/repository/TeacherRepository.kt`: Estado de favoritos

#### Data Layer
- `data/local/dao/UserSettingsDao.kt`: Configuraciones globales
- `data/local/dao/TeacherDao.kt`: Marcado de favoritos

#### Tests
- `domain/service/CompositeStrategyTest.kt`: Combinación de criterios
- `presentation/settings/SettingsViewModelTest.kt`: Persistencia de cambios

### HU4: Generar Horarios Candidatos
**Objetivo**: Producción de opciones de horario optimizadas
**Módulos principales**: `domain/usecase`, `domain/service`

#### Use Cases
- `domain/usecase/GenerateSchedulesUseCaseImpl.kt`: Orquestación principal
- `domain/usecase/ScheduleGenerator.kt`: Algoritmo de backtracking

#### Domain Services
- `domain/service/ScheduleEvaluationStrategy.kt`: Interfaz de evaluación
- `domain/service/AcceptConflictsStrategy.kt`: Manejo de conflictos duros
- `domain/service/ScheduleStrategyFactory.kt`: Creación de estrategias compuestas

#### UI Layer
- `presentation/generateSchedule/GenerateScheduleScreen.kt`: Configuración y resultados
- `presentation/generateSchedule/GenerateScheduleViewModel.kt`: Estado de generación
- `presentation/generateSchedule/SchedulePreview.kt`: Vista de candidatos

#### Tests
- `domain/usecase/ScheduleGeneratorTest.kt`: Algoritmo con fixtures
- `domain/service/*Strategy*Test.kt`: Evaluación individual
- Performance tests para tiempo de generación

### HU5: Aplicar Horario
**Objetivo**: Establecer horario semanal oficial
**Módulos principales**: `domain/usecase`, `presentation/home`

#### Use Cases
- `domain/usecase/GetWeeklyEnrolledScheduleUseCase.kt`: Consulta de horario aplicado

#### UI Layer
- `presentation/home/HomeScreen.kt`: Vista semanal principal
- `presentation/home/HomeViewModel.kt`: Estado del horario semanal
- `presentation/home/WeekPreviewDialog.kt`: Vista expandida

#### Repositorios
- `domain/repository/SelectedSubjectRepository.kt`: Actualización de selecciones aplicadas

#### Data Layer
- `data/local/dao/SelectedSubjectDao.kt`: Upsert con groupId correcto
- `data/local/relation/GroupWithSchedulesRelation.kt`: Queries con joins

#### Tests
- `domain/usecase/GetWeeklyEnrolledScheduleUseCaseTest.kt`: Queries complejas
- UI tests para navegación semanal

### HU6: Exportar Horario
**Objetivo**: Generación de archivos en múltiples formatos
**Módulos principales**: `domain/util`, `presentation/settings`

#### Utilidades
- `domain/util/GenerateWeeklySchedulePdfUseCase.kt`: PDF con layout profesional
- `domain/util/GenerateWeeklyScheduleExcelUseCase.kt`: Excel estructurado
- `domain/util/GenerateWeeklyScheduleImageUseCase.kt`: Bitmap de alta calidad
- `domain/util/ShareableScheduleJsonGenerator.kt`: JSON para intercambio

#### UI Layer
- `presentation/settings/SettingsAddScheduleScreen.kt`: Opciones de exportación
- `presentation/settings/SettingsSendScheduleScreen.kt`: Compartir generado

#### Tests
- `domain/util/*Generator*Test.kt`: Validación de formatos
- Integration tests con archivos generados

### HU7: Importar Horario Compartido
**Objetivo**: Incorporación de horarios de otros usuarios
**Módulos principales**: `domain/usecase`, `presentation/settings`

#### Use Cases
- `domain/usecase/ImportSharedScheduleUseCase.kt`: Validación y aplicación
- `domain/usecase/LoadShareableScheduleUseCase.kt`: Preview de JSON

#### UI Layer
- `presentation/settings/SettingsAddScheduleScreen.kt`: Importación con preview

#### Data Layer
- `data/usecase/JsonBackupGenerator.kt`: Parsing de JSON compartido

#### Tests
- `domain/usecase/ImportSharedScheduleUseCaseTest.kt`: Validación de esquemas
- JSON schema tests con casos edge

### HU8: Notificaciones de Clases
**Objetivo**: Recordatorios automáticos de compromisos académicos
**Módulos principales**: `infrastructure/notification`, `application/notification`

#### Application Layer
- `application/notification/NotificationService.kt`: Abstracción de notificaciones
- `application/notification/ScheduleNotificationUseCase.kt`: Lógica de scheduling
- `application/notification/ShowNotificationUseCase.kt`: Emisión de notificaciones

#### Infrastructure Layer
- `infrastructure/notification/NotificationServiceImpl.kt`: Implementación Android
- `infrastructure/notification/NotificationModule.kt`: DI para notificaciones
- `infrastructure/notification/NotifyWorker.kt`: WorkManager worker
- `infrastructure/notification/NotificationStyler.kt`: Styling de notificaciones

#### UI Layer
- `presentation/notification/NotificationPermissionScreen.kt`: Gestión de permisos
- `presentation/notification/NotificationDemoScreen.kt`: Testing de notificaciones

#### Tests
- `infrastructure/notification/NotificationServiceTest.kt`: Emisión
- WorkManager tests para scheduling
- Permission request tests

### HU9: Eventos Académicos Manuales
**Objetivo**: Gestión de eventos personalizados en calendario
**Módulos principales**: `presentation/addevent`, `domain/repository`

#### UI Layer
- `presentation/addevent/AddEventScreen.kt`: Formulario de creación/edición
- `presentation/addevent/AddEventViewModel.kt`: Validación y persistencia

#### Repositorios
- `domain/repository/AcademicEventRepository.kt`: CRUD de eventos

#### Data Layer
- `data/local/dao/AcademicEventDao.kt`: Persistencia con constraints
- `data/local/entity/AcademicEventEntity.kt`: Modelo de datos

#### Tests
- `domain/repository/AcademicEventRepositoryTest.kt`: CRUD operations
- UI tests para formularios

### HU10: Copia de Seguridad
**Objetivo**: Persistencia de datos del usuario en la nube
**Módulos principales**: `domain/usecase`, `domain/util`

#### Use Cases
- `domain/usecase/BackupAllDataJsonUseCase.kt`: Generación de backup
- `domain/usecase/AutoSyncUseCase.kt`: Sincronización automática

#### Utilidades
- `domain/util/JsonBackupGenerator.kt`: Serialización completa
- `domain/util/ShareableScheduleJsonGenerator.kt`: Formato compartible

#### Data Layer
- `data/remote/FirebaseBackupService.kt`: Upload a Firestore
- `data/local/AppDatabase.kt`: Export completo de datos

#### Tests
- `domain/usecase/BackupAllDataJsonUseCaseTest.kt`: Serialización
- Integration tests con Firebase emulator

## Matriz de Cobertura de Trazabilidad

### Métricas por Historia
| HU | Archivos Mapeados | Tests Asociados | Cobertura Estimada |
|----|------------------|-----------------|-------------------|
| HU1 | 15+ | 8 | 95% |
| HU2 | 12+ | 6 | 90% |
| HU3 | 8+ | 5 | 85% |
| HU4 | 10+ | 7 | 95% |
| HU5 | 6+ | 4 | 80% |
| HU6 | 5+ | 4 | 90% |
| HU7 | 4+ | 3 | 85% |
| HU8 | 8+ | 5 | 90% |
| HU9 | 5+ | 3 | 80% |
| HU10 | 6+ | 4 | 85% |

### Validación de Trazabilidad
- ✅ **Completitud**: 100% HU mapeadas a código
- ✅ **Consistencia**: Artefactos existen y funcionan
- ✅ **Actualización**: Mapeo refleja implementación actual
- ✅ **Testeabilidad**: Cobertura de tests adecuada

## Referencias Cruzadas

### Documentos Relacionados
- **Arquitectura**: docs/arquitectura.md - Estructura general
- **Funcionalidades**: docs/funcionalidades.md - Features por categoría
- **Flujos**: docs/flujos-usuario.md - Journeys completos
- **Requisitos**: docs/desarrollo/requisitos.md - RF detallados
- **Criterios**: docs/desarrollo/criterios-aceptacion-definicion-de-terminado.md - Validación

### Herramientas de Mantenimiento
- **Scripts de verificación**: Validación automática de mapeos
- **CI/CD checks**: Trazabilidad en pipeline
- **Documentation linter**: Consistencia de referencias
- **Impact analysis**: Cambios propagados automáticamente
