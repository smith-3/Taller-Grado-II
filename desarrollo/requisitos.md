# Requisitos del Proyecto TecnoTime

## Metodología de Captura de Requisitos

### Enfoque Híbrido
- **Análisis inicial**: Levantamiento de necesidades basado en problemática universitaria
- **Iterativo**: Refinamiento continuo basado en prototipado y feedback
- **Validación**: Verificación contra código fuente y casos de uso reales
- **Trazabilidad**: Mapping completo RF → HU → código → tests

### Clasificación
- **Funcionales (RF)**: Comportamientos observables del sistema
- **No Funcionales (RNF)**: Características de calidad y restricciones
- **Supuestos**: Condiciones externas asumidas
- **Dependencias**: Requisitos de terceros sistemas

## Requisitos Funcionales (RF)

### RF1: Adquisición de Datos Académicos
**Descripción**: Sistema de scraping automático para obtener oferta académica completa desde UMSS
**Subrequisitos**:
- RF1.1: Descubrimiento automático de endpoints scrapeables
- RF1.2: Parsing robusto de PDFs de horarios con expresiones regulares configurables
- RF1.3: Manejo de cambios en estructura HTML/PDF con fallbacks
- RF1.4: Validación semántica de datos parseados (horarios 07:00-22:00)
- RF1.5: Logging detallado para debugging y monitoreo

**Criterios de Aceptación**:
- ✅ ≥90% de bloques horarios válidos parseados correctamente
- ✅ Recuperación graceful de errores de parsing
- ✅ Configuración externa de patrones regex
- ✅ Telemetría de éxito/error por fuente

### RF2: Persistencia de Datos
**Descripción**: Base de datos local normalizada con integridad referencial completa
**Subrequisitos**:
- RF2.1: Modelo entidad-relación con constraints de integridad
- RF2.2: Transacciones ACID para operaciones complejas
- RF2.3: Migraciones versionadas para evolución de schema
- RF2.4: Índices optimizados para queries de jerarquía
- RF2.5: Validaciones a nivel de base de datos

**Criterios de Aceptación**:
- ✅ Foreign keys activas en todas las relaciones
- ✅ Rollback automático en operaciones fallidas
- ✅ Queries complejas <100ms en dispositivo típico
- ✅ Soporte para datos históricos y auditoría

### RF3: Selección Jerárquica de Materias
**Descripción**: Interfaz intuitiva para navegación carrera→nivel→materia→grupo
**Subrequisitos**:
- RF3.1: Estado UI consistente durante navegación
- RF3.2: Preview de conflictos en tiempo real
- RF3.3: Persistencia de selecciones parciales
- RF3.4: Validación de prerrequisitos académicos
- RF3.5: Búsqueda y filtrado de opciones

**Criterios de Aceptación**:
- ✅ Navegación fluida sin pérdida de contexto
- ✅ Estado sobrevive rotación de dispositivo
- ✅ Feedback visual inmediato de conflictos
- ✅ Performance lineal con tamaño de datos

### RF4: Generación Inteligente de Horarios
**Descripción**: Algoritmo de optimización combinatoria con estrategias configurables
**Subrequisitos**:
- RF4.1: Backtracking completo del espacio de combinaciones
- RF4.2: Poda temprana para eficiencia computacional
- RF4.3: Estrategias de evaluación ponderadas (gaps, docentes, días)
- RF4.4: Restricciones duras (no solapamientos) vs blandas (preferencias)
- RF4.5: Timeout configurable para casos patológicos

**Criterios de Aceptación**:
- ✅ Generación <3s para casos típicos (6×3 combinaciones)
- ✅ Al menos 3 candidatos viables en >95% casos
- ✅ Score determinístico y reproducible
- ✅ Memoria bounded (no leaks en ejecuciones largas)

### RF5: Configuración de Preferencias
**Descripción**: Sistema granular de ajustes para personalización de comportamiento
**Subrequisitos**:
- RF5.1: Pesos ajustables para criterios de evaluación
- RF5.2: Gestión de docentes favoritos
- RF5.3: Configuración de formato hora (12/24h)
- RF5.4: Preferencias de notificaciones por materia
- RF5.5: Reset a valores por defecto

**Criterios de Aceptación**:
- ✅ Persistencia inmediata de cambios
- ✅ Validación de rangos de valores
- ✅ Preview de impacto en score
- ✅ Sincronización entre sesiones

### RF6: Visualización de Horarios
**Descripción**: Interfaces ricas para vista semanal y candidatos
**Subrequisitos**:
- RF6.1: Calendario semanal interactivo
- RF6.2: Detalle expandible de materias/eventos
- RF6.3: Indicadores visuales de conflictos
- RF6.4: Navegación temporal (semanas previas/siguientes)
- RF6.5: Modo comparación entre candidatos

**Criterios de Aceptación**:
- ✅ Renderizado fluido en 60fps
- ✅ Zoom y scroll naturales
- ✅ Información contextual on-demand
- ✅ Accesibilidad completa (screen readers)

### RF7: Exportación Multiformal
**Descripción**: Generación de archivos en múltiples formatos para distribución
**Subrequisitos**:
- RF7.1: PDF con layout profesional
- RF7.2: Excel con datos estructurados
- RF7.3: Imagen bitmap de alta calidad
- RF7.4: JSON para intercambio programático
- RF7.5: Nombres de archivo consistentes con timestamp

**Criterios de Aceptación**:
- ✅ Archivos válidos abribles en aplicaciones estándar
- ✅ Contenido completo y legible
- ✅ Generación asíncrona sin bloqueo UI
- ✅ Progreso visible para archivos grandes

### RF8: Importación y Compartir
**Descripción**: Sistema de intercambio de horarios entre usuarios
**Subrequisitos**:
- RF8.1: Validación de esquema JSON
- RF8.2: Preview antes de importación
- RF8.3: Merge vs replace completo
- RF8.4: Deep linking para compartir
- RF8.5: Historial de importaciones

**Criterios de Aceptación**:
- ✅ Validación robusta contra datos malformados
- ✅ Preview completo con estadísticas
- ✅ Operaciones atómicas (rollback en error)
- ✅ Integración nativa con sharing del sistema

### RF9: Sistema de Notificaciones
**Descripción**: Recordatorios inteligentes de clases y eventos
**Subrequisitos**:
- RF9.1: Permisos Android gestionados correctamente
- RF9.2: Scheduling preciso con WorkManager
- RF9.3: Acciones en notificaciones (posponer, descartar)
- RF9.4: Personalización por materia
- RF9.5: Respeta Do Not Disturb

**Criterios de Aceptación**:
- ✅ Entrega >95% en condiciones normales
- ✅ Timing preciso (±1 minuto)
- ✅ Batería eficiente (<5% consumo diario)
- ✅ Funciona con app cerrada

### RF10: Sincronización y Backup
**Descripción**: Persistencia de datos del usuario en la nube
**Subrequisitos**:
- RF10.1: Backup automático periódico
- RF10.2: Restore con confirmación
- RF10.3: Encriptación de datos sensibles
- RF10.4: Versioning para evolución de datos
- RF10.5: Conflicto resolution en sync

**Criterios de Aceptación**:
- ✅ Backup completo en <30 segundos
- ✅ Restore íntegro con verificación
- ✅ Encriptación AES-256
- ✅ Resolución automática de conflictos

### RF11: Gestión de Eventos Académicos
**Descripción**: CRUD completo para eventos personalizados
**Subrequisitos**:
- RF11.1: Creación con validación de campos
- RF11.2: Detección automática de conflictos
- RF11.3: Categorización (examen, reunión, tarea)
- RF11.4: Notificaciones programadas
- RF11.5: Integración con calendario semanal

**Criterios de Aceptación**:
- ✅ Validación completa de entrada
- ✅ Detección >90% accuracy de conflictos
- ✅ Persistencia transaccional
- ✅ Notificaciones independientes del horario

### RF12: Sistema de Docentes Favoritos
**Descripción**: Marcado y consideración de preferencias docentes
**Subrequisitos**:
- RF12.1: UI de toggle intuitiva
- RF12.2: Integración con algoritmo de generación
- RF12.3: Persistencia en perfil docente
- RF12.4: Estadísticas de uso
- RF12.5: Reset masivo si necesario

**Criterios de Aceptación**:
- ✅ Estado consistente en todas las vistas
- ✅ Impacto directo en score de generación
- ✅ Sincronización con datos de scraping
- ✅ Performance no degradada

### RF13: Configuración Avanzada
**Descripción**: Ajustes granulares del comportamiento de la aplicación
**Subrequisitos**:
- RF13.1: Formato hora 12/24h global
- RF13.2: Pesos de estrategias de generación
- RF13.3: Notificaciones por materia
- RF13.4: Tema de la aplicación
- RF13.5: Configuración de idioma

**Criterios de Aceptación**:
- ✅ Aplicación inmediata de cambios
- ✅ Validación de combinaciones inválidas
- ✅ Persistencia en UserSettings
- ✅ Sincronización entre dispositivos

### RF14: Navegación y Detalle
**Descripción**: Sistema de navegación rico con contexto detallado
**Subrequisitos**:
- RF14.1: Deep linking a estados específicos
- RF14.2: Breadcrumb navigation
- RF14.3: Historial de navegación
- RF14.4: Context menus apropiados
- RF14.5: Shortcuts de teclado

**Criterios de Aceptación**:
- ✅ Navegación intuitiva sin back button overuse
- ✅ Estado contextual preservado
- ✅ Performance de navegación <100ms
- ✅ Accesibilidad completa

### RF15: Resiliencia de Red
**Descripción**: Manejo robusto de conectividad intermitente
**Subrequisitos**:
- RF15.1: Reintentos con backoff exponencial
- RF15.2: Timeouts configurables
- RF15.3: Cache offline inteligente
- RF15.4: Feedback de estado de conexión
- RF15.5: Queue de operaciones pendientes

**Criterios de Aceptación**:
- ✅ Recuperación automática de fallos temporales
- ✅ Estado offline funcional
- ✅ Feedback claro al usuario
- ✅ No pérdida de datos en desconexiones

## Requisitos No Funcionales (RNF)

### RNF1: Performance
**Métricas Objetivo**:
- Generación de horarios: <3s (P95) para 6×3 combinaciones
- Inicio de app: <2s cold start
- Queries de base de datos: <100ms P95
- UI rendering: 60fps sostenido
- Memoria: <100MB heap usage normal

### RNF2: Persistencia y Consistencia
- **ACID**: Todas las operaciones transaccionales
- **Integridad**: Constraints activos en todas las relaciones
- **Durabilidad**: Datos sobreviven crashes y reinicios
- **Consistencia**: Estado válido en todo momento

### RNF3: Disponibilidad Offline
- **Funcionalidad core**: 100% operable sin red
- **Sincronización**: Background automática al reconectar
- **Cache**: Datos suficientes para 30 días típicos
- **Feedback**: Indicadores claros de estado offline

### RNF4: Usabilidad y Accesibilidad
- **Accesibilidad AA**: Contraste 4.5:1, tamaños ≥48dp
- **Internacionalización**: Strings externalizados, RTL support
- **Navegación**: Keyboard completa, screen readers
- **Feedback**: Estados de carga, errores claros, progreso

### RNF5: Seguridad
- **Encriptación**: Datos sensibles en AES-256
- **Permisos**: Mínimos necesarios, justificación clara
- **Input validation**: Sanitización completa
- **Auditoría**: Logs de operaciones sensibles

### RNF6: Mantenibilidad
- **Arquitectura**: Clean Architecture estricta
- **Modularidad**: Separación clara de responsabilidades
- **Documentación**: Código autodocumentado, docs actualizadas
- **Testing**: Cobertura 70%+, tests automatizados

### RNF7: Observabilidad
- **Logging**: Estructurado con niveles apropiados
- **Métricas**: Performance, uso, errores
- **Monitoring**: Crash reporting, analytics
- **Debugging**: Información rica en errores

### RNF8: Compatibilidad
- **Android**: API 24+ (7.0) a 35+ (15)
- **Dispositivos**: 95% cobertura de mercado
- **Resolución**: 320dp+ width, adaptive layouts
- **Orientación**: Portrait/landscape support

### RNF9: Escalabilidad
- **Datos**: Soporte para 100+ carreras, 1000+ materias
- **Usuarios**: Arquitectura preparada para sync
- **Performance**: Algoritmos O(n log n) o mejor
- **Almacenamiento**: <500MB para datos completos

### RNF10: Eficiencia Energética
- **Batería**: <5% consumo diario típico
- **Trabajos**: Batch processing, constraints apropiadas
- **Optimizaciones**: Wake locks mínimos, efficient polling

## Supuestos y Dependencias

### Supuestos del Sistema
- **UMSS website**: Accesible y relativamente estable en estructura
- **Conectividad**: Intermitente pero disponible para sincronización inicial
- **Permisos**: Usuario otorga permisos necesarios (notificaciones, storage)
- **Dispositivo**: Android moderno con capacidades estándar
- **Datos**: Información académica pública y ética para scraping

### Dependencias Externas
- **Firebase**: Auth + Firestore para backup (gratuito tier)
- **Google Play Services**: Para notificaciones y backup
- **UMSS website**: Disponibilidad 99%+ para scraping
- **Android Framework**: APIs estables en versiones soportadas

## Matriz de Trazabilidad Requisitos → Artefactos

### Mapeo Funcional → Código
| Requisito | Artefactos Principales | Validación |
|-----------|----------------------|------------|
| RF1 | `domain/usecase/FetchCareersUseCase.kt`, `infrastructure/scraping/*` | Tests de integración scraping |
| RF2 | `data/local/*`, `AppDatabase.kt` | Tests de Room, migraciones |
| RF3 | `presentation/addsubject/*`, `presentation/careerselection/*` | UI tests, state preservation |
| RF4 | `domain/usecase/ScheduleGenerator.kt`, `domain/service/*Strategy*` | Unit tests algoritmos, performance tests |
| RF5 | `presentation/settings/*`, `domain/repository/UserSettingsRepository.kt` | Integration tests settings |
| RF6 | `presentation/home/*`, `presentation/generateSchedule/*` | UI tests, accessibility tests |
| RF7 | `domain/util/*Generator*.kt` | File generation tests, format validation |
| RF8 | `domain/usecase/ImportSharedScheduleUseCase.kt` | JSON schema tests, import validation |
| RF9 | `infrastructure/notification/*`, `application/notification/*` | Notification tests, WorkManager tests |
| RF10 | `domain/usecase/BackupAllDataJsonUseCase.kt` | Backup/restore integration tests |
| RF11 | `presentation/addevent/*`, `domain/repository/AcademicEventRepository.kt` | CRUD tests, conflict detection |
| RF12 | `presentation/teachers/*`, `domain/service/PrioritizeTeachersStrategy.kt` | Strategy tests, UI state tests |
| RF13 | `domain/repository/UserSettingsRepository.kt` | Settings persistence tests |
| RF14 | `presentation/navigation/*`, `ui/common/*` | Navigation tests, deep link tests |
| RF15 | `domain/usecase/*Sync*.kt`, `infrastructure/network/*` | Network resilience tests |

### Métricas de Cumplimiento
- **Cobertura**: 100% requisitos funcionales mapeados a código
- **Testeabilidad**: 95% requisitos con tests automatizados
- **Trazabilidad**: Bidireccional RF ↔ código ↔ tests
- **Mantenibilidad**: Documentación actualizada con cambios de código
