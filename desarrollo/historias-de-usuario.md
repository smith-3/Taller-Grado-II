# Historias de Usuario (HU) y Criterios de Aceptación

## Metodología de Definición

### Formato INVEST
Todas las HU siguen el formato INVEST:
- **I**ndependent: Independientes entre sí
- **N**egotiable: Abiertas a discusión y refinamiento
- **V**aluable: Entregan valor claro al usuario
- **E**stimable: Posibles de estimar en esfuerzo
- **S**mall: Completables en una iteración
- **T**estable: Verificables con criterios claros

### Priorización MoSCoW
- **Must Have**: Funcionalidad core (HU1-HU5)
- **Should Have**: Features importantes (HU6-HU8)
- **Could Have**: Mejoras de UX (HU9-HU10)
- **Won't Have**: Fuera de scope inicial

### Criterios de Aceptación
Cada HU incluye:
- **Given-When-Then**: Escenario BDD completo
- **Validación técnica**: Requisitos no funcionales
- **Casos edge**: Manejo de errores y casos límite
- **Métricas**: KPIs para medir éxito

## Historias de Usuario Detalladas

### HU1: Sincronizar Oferta Académica
**Como** estudiante de la UMSS
**Quiero** sincronizar carreras, niveles, materias y grupos
**Para** disponer de datos actualizados para generar horarios

#### Escenario Principal: Primera Sincronización
**Dado que** soy un estudiante nuevo en la app
**Cuando** selecciono una carrera de la lista
**Entonces** veo el progreso de sincronización y accedo a niveles/materias

#### Criterios de Aceptación
- ✅ **Lista de carreras**: Mínimo 5 carreras válidas cargadas desde UMSS
- ✅ **Progreso visual**: Indicadores de carga durante scraping de niveles→materias→grupos
- ✅ **Persistencia**: Datos guardados localmente tras sincronización exitosa
- ✅ **Feedback**: Mensaje de éxito y navegación automática a selección de niveles
- ✅ **Reintento**: Máximo 3 intentos automáticos en errores de red
- ✅ **Offline**: Estado preservado si falla conectividad

#### Validación Técnica
- ✅ Parsing PDF ≥90% de bloques horarios válidos
- ✅ Transacciones Room para integridad referencial
- ✅ Logging detallado para debugging de scraping

#### Casos Edge
- **Red lenta**: Timeout apropiado con mensaje de reintento
- **PDF corrupto**: Fallback a datos previos con warning
- **Carrera sin niveles**: UI muestra mensaje explicativo

#### Métricas de Éxito
- Tiempo promedio sincronización: <30 segundos
- Tasa éxito: >95% en conexiones estables
- Retención: >80% usuarios completan sincronización

### HU2: Seleccionar Materias y Grupos
**Como** estudiante con carrera sincronizada
**Quiero** elegir materias y grupos específicos
**Para** personalizar mi horario académico

#### Escenario Principal: Selección Jerárquica
**Dado que** tengo niveles cargados
**Cuando** navego carrera→nivel→materia→grupo
**Entonces** puedo seleccionar y personalizar mis elecciones

#### Criterios de Aceptación
- ✅ **Jerarquía completa**: Navegación fluida entre todos los niveles
- ✅ **Múltiples selecciones**: Varios grupos por materia permitidos
- ✅ **Preview visual**: Vista de conflictos en tiempo real
- ✅ **Personalización**: Colores, emojis, configuración de notificaciones
- ✅ **Persistencia**: Selecciones sobreviven navegación y reinicio
- ✅ **Validación**: Detección básica de conflictos horarios

#### Validación Técnica
- ✅ StateFlow reactivos para UI consistente
- ✅ SavedStateHandle para supervivencia de configuración
- ✅ Queries Room eficientes para jerarquía

#### Casos Edge
- **Grupos agotados**: Mensaje claro cuando no hay cupo
- **Horarios conflictivos**: Highlighting visual de solapamientos
- **Selección vacía**: Validación antes de proceder

#### Métricas de Éxito
- Completion rate: >90% usuarios completan selección
- Tiempo promedio: <5 minutos para selección típica
- Error rate: <5% en validaciones

### HU3: Preferencias de Generación
**Como** estudiante con selecciones realizadas
**Quiero** configurar criterios de optimización
**Para** obtener horarios que se ajusten a mis prioridades

#### Escenario Principal: Configuración de Estrategias
**Dado que** tengo materias seleccionadas
**Cuando** ajusto pesos y favoritos
**Entonces** veo cómo afectan al score de generación

#### Criterios de Aceptación
- ✅ **Controles intuitivos**: Sliders y toggles para pesos de estrategias
- ✅ **Docentes favoritos**: Toggle visual para marcar preferencias
- ✅ **Preview de impacto**: Score actualizado en tiempo real
- ✅ **Persistencia**: Configuraciones guardadas en UserSettings
- ✅ **Reset**: Opción de volver a valores por defecto

#### Validación Técnica
- ✅ CompositeStrategy combina criterios correctamente
- ✅ Pesos aplicados en cálculo de score
- ✅ UI refleja cambios sin lag

#### Casos Edge
- **Sin favoritos**: Algoritmo funciona con pesos por defecto
- **Pesos extremos**: Validación de valores razonables
- **Cambio durante generación**: Recálculo automático

#### Métricas de Éxito
- Config completion: >85% usuarios ajustan al menos un criterio
- Satisfaction: >4/5 en facilidad de configuración

### HU4: Generar Horarios Candidatos
**Como** estudiante con configuración completa
**Quiero** generar múltiples opciones de horario
**Para** elegir la que mejor se adapte a mis necesidades

#### Escenario Principal: Generación Óptima
**Dado que** tengo selecciones y preferencias
**Cuando** ejecuto la generación
**Entonces** obtengo candidatos ordenados por calidad

#### Criterios de Aceptación
- ✅ **Múltiples candidatos**: Al menos 3 opciones viables
- ✅ **Sin conflictos duros**: Ningún solapamiento de horarios
- ✅ **Orden por score**: Mejor candidato primero
- ✅ **Métricas visibles**: Gaps, días utilizados, docentes favoritos
- ✅ **Performance**: Generación <3 segundos para casos típicos
- ✅ **Regeneración**: Opción de nuevos candidatos

#### Validación Técnica
- ✅ Backtracking completo del espacio combinatorio
- ✅ Poda temprana para eficiencia
- ✅ Score finito para candidatos válidos

#### Casos Edge
- **Espacio pequeño**: Generación instantánea con pocos candidatos
- **Sin solución**: Mensaje claro cuando no hay combinaciones válidas
- **Timeout**: Cancelación graceful con resultados parciales

#### Métricas de Éxito
- Success rate: >95% generan al menos un candidato válido
- Satisfaction: >4.5/5 en calidad de resultados
- Performance: P95 <5 segundos

### HU5: Aplicar Horario
**Como** estudiante con candidatos generados
**Quiero** seleccionar y aplicar un horario
**Para** tener mi horario semanal oficial

#### Escenario Principal: Aplicación Definitiva
**Dado que** tengo candidatos generados
**Cuando** selecciono y aplico uno
**Entonces** veo mi horario semanal activo

#### Criterios de Aceptación
- ✅ **Vista semanal poblada**: Horario completo visible
- ✅ **Navegación a detalle**: Acceso a información de materias
- ✅ **Persistencia**: Horario sobrevive reinicio app
- ✅ **Notificaciones activas**: Recordatorios programados
- ✅ **Modificación**: Opción de cambiar horario aplicado

#### Validación Técnica
- ✅ SelectedSubject actualizado con groupId correcto
- ✅ WorkManager programa notificaciones
- ✅ Relaciones de base de datos consistentes

#### Casos Edge
- **Cambio de horario**: Cleanup de notificaciones previas
- **Grupos no disponibles**: Validación antes de aplicación
- **Aplicación parcial**: Rollback en caso de error

#### Métricas de Éxito
- Application rate: >70% usuarios aplican horario generado
- Retention: >90% mantienen horario aplicado >1 semana

### HU6: Exportar Horario
**Como** estudiante con horario aplicado
**Quiero** exportar mi horario en múltiples formatos
**Para** compartir o imprimir

#### Escenario Principal: Exportación Completa
**Dado que** tengo horario semanal
**Cuando** solicito exportación
**Entonces** obtengo archivos en formatos solicitados

#### Criterios de Aceptación
- ✅ **Formatos múltiples**: PDF, Excel, imagen, JSON
- ✅ **Contenido completo**: Todo el horario legible
- ✅ **Nombres consistentes**: Timestamp + formato descriptivo
- ✅ **Sharing nativo**: Integración con intents del sistema
- ✅ **Preview**: Vista previa antes de generación

#### Validación Técnica
- ✅ Generadores especializados por formato
- ✅ Manejo eficiente de memoria
- ✅ Encoding correcto para caracteres especiales

#### Casos Edge
- **Horario vacío**: Mensaje apropiado sin generación
- **Permisos denegados**: Request y retry
- **Errores de formato**: Fallback a JSON básico

#### Métricas de Éxito
- Export rate: >60% usuarios exportan al menos una vez
- Success rate: >98% exportaciones exitosas
- Format preference: PDF > Excel > Imagen

### HU7: Importar Horario Compartido
**Como** estudiante
**Quiero** importar horarios compartidos por compañeros
**Para** usar como base o referencia

#### Escenario Principal: Importación Validada
**Dado que** recibo archivo JSON de horario
**Cuando** importo el archivo
**Entonces** veo preview y aplico de forma controlada

#### Criterios de Aceptación
- ✅ **Validación de esquema**: Estructura JSON correcta verificada
- ✅ **Preview detallada**: Vista completa del horario a importar
- ✅ **Opciones de fusión**: Merge vs replace completo
- ✅ **Aplicación segura**: Sin corrupción de datos existentes
- ✅ **Feedback**: Estadísticas de importación exitosa

#### Validación Técnica
- ✅ Parsing JSON robusto con error handling
- ✅ Validación semántica de referencias
- ✅ Transacciones para atomicidad

#### Casos Edge
- **Schema obsoleto**: Mensaje de incompatibilidad
- **Datos corruptos**: Rechazo con explicación
- **Conflictos**: Opciones de resolución

#### Métricas de Éxito
- Import rate: >30% usuarios importan al menos una vez
- Success rate: >90% importaciones exitosas

### HU8: Notificaciones de Clases
**Como** estudiante con horario aplicado
**Quiero** recibir recordatorios de clases
**Para** no perder mis compromisos académicos

#### Escenario Principal: Recordatorio Oportuno
**Dado que** tengo clase en 10 minutos
**Cuando** se acerca la hora
**Entonces** recibo notificación con acciones

#### Criterios de Aceptación
- ✅ **Timing preciso**: 5-15 minutos antes (configurable)
- ✅ **Información completa**: Materia, aula, docente
- ✅ **Acciones**: Posponer 5 min, descartar
- ✅ **Personalización**: Sonido/vibración según preferencias
- ✅ **Do Not Disturb**: Respeta configuración del sistema

#### Validación Técnica
- ✅ WorkManager con constraints de batería
- ✅ AlarmManager para precisión de timing
- ✅ PendingIntents seguros

#### Casos Edge
- **Dispositivo apagado**: Notificación al despertar
- **App cerrada**: Notificaciones siguen funcionando
- **Zona horaria**: Ajuste automático

#### Métricas de Éxito
- Delivery rate: >95% notificaciones entregadas
- Interaction rate: >50% usuarios interactúan con notificaciones
- Satisfaction: >4/5 en utilidad de recordatorios

### HU9: Eventos Académicos Manuales
**Como** estudiante
**Quiero** agregar eventos personalizados al calendario
**Para** gestionar exámenes, reuniones y tareas

#### Escenario Principal: CRUD de Eventos
**Dado que** quiero agregar un examen
**Cuando** creo el evento
**Entonces** aparece en el calendario con validaciones

#### Criterios de Aceptación
- ✅ **CRUD completo**: Crear, leer, actualizar, eliminar
- ✅ **Validación**: Campos requeridos y formatos correctos
- ✅ **Visualización**: Eventos en vista semanal
- ✅ **Detección de conflictos**: Alertas de solapamientos
- ✅ **Categorización**: Tipos (examen, reunión, tarea)
- ✅ **Notificaciones**: Recordatorios programados

#### Validación Técnica
- ✅ AcademicEventEntity con relaciones correctas
- ✅ Validación de solapamientos en dominio
- ✅ Cascade operations para integridad

#### Casos Edge
- **Eventos recurrentes**: Soporte básico semanal
- **Conflictos múltiples**: Resolución por prioridad
- **Edición masiva**: No permitida inicialmente

#### Métricas de Éxito
- Usage rate: >40% usuarios crean al menos un evento
- Conflict detection: >90% accuracy

### HU10: Copia de Seguridad
**Como** estudiante
**Quiero** respaldar y restaurar mis datos
**Para** no perder mi configuración y selecciones

#### Escenario Principal: Backup Completo
**Dado que** tengo datos en la app
**Cuando** ejecuto backup
**Entonces** mis datos quedan seguros en la nube

#### Criterios de Aceptación
- ✅ **Backup completo**: Todos los datos serializados
- ✅ **Almacenamiento seguro**: Firebase Storage encriptado
- ✅ **Restore validado**: Confirmación y preview
- ✅ **Integridad**: Verificación post-restore
- ✅ **Progreso**: Indicadores para operaciones largas

#### Validación Técnica
- ✅ Serialización completa del grafo de objetos
- ✅ Encriptación de datos sensibles
- ✅ Versioning para compatibilidad futura

#### Casos Edge
- **Espacio insuficiente**: Mensaje claro y opciones
- **Restore fallido**: Rollback automático
- **Versiones incompatibles**: Mensaje de upgrade requerido

#### Métricas de Éxito
- Backup rate: >50% usuarios hacen backup regular
- Restore success: >95% restores exitosos
- Data integrity: 100% verificada

## Referencias Cruzadas

### Documentos Relacionados
- **Flujos de usuario**: docs/flujos-usuario.md - Journeys completos
- **Algoritmos**: docs/algoritmos-generacion-horarios.md - Lógica detrás HU4
- **Marco teórico**: docs/marco_teorico.md - Fundamentos científicos
- **Criterios de aceptación**: criterios-aceptacion-definicion-de-terminado.md - Validación detallada
- **Trazabilidad**: trazabilidad.md - HU → código mapping

### Métricas Globales de HU
- **Coverage**: 100% funcionalidades core cubiertas
- **Testability**: Todas las HU tienen criterios verificables
- **Value delivery**: Cada HU entrega valor tangible al usuario
- **Independence**: Mínima dependencia entre historias
- **Estimability**: Todas estimables en story points

### Evolución del Backlog
- **Refinement continuo**: HU refinadas basado en feedback
- **Priorización dinámica**: Ajuste según aprendizaje del desarrollo
- **Split de historias**: HU complejas divididas cuando necesario
- **Épicas emergentes**: Nuevas HU identificadas durante desarrollo
