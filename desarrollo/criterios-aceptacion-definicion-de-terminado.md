# Criterios de Aceptación y Definición de Terminado (DoD)

## Definición de Terminado (DoD) Global

### Criterios Técnicos Obligatorios
- ✅ **Compilación exitosa**: Build `release` sin errores/warnings críticos
- ✅ **Funcionalidad navegable**: Flujo completo operable en dispositivo/emulador
- ✅ **Validación de datos**: Lógica probada con datos reales o fixtures representativos
- ✅ **Estabilidad**: Sin crashes en acciones principales (0 crashes en testing)
- ✅ **Documentación**: README y docs relacionados actualizados y precisos

### Criterios de Calidad
- ✅ **Testing**: Cobertura mínima 70% en lógica crítica, tests automatizados pasan
- ✅ **Performance**: Operaciones principales <3 segundos, UI fluida (60fps)
- ✅ **Accesibilidad**: Cumple WCAG AA básico, navegación por teclado funcional
- ✅ **Internacionalización**: Strings externalizados, soporte básico multiidioma preparado
- ✅ **Seguridad**: Datos sensibles encriptados, permisos mínimos necesarios

### Criterios de Producto
- ✅ **UX completa**: Feedback visual, estados de carga, manejo de errores
- ✅ **Persistencia**: Datos sobreviven reinicio app y actualización
- ✅ **Offline**: Funcionalidad core operable sin conectividad
- ✅ **Compatibilidad**: Android 7.0+ (API 24), dispositivos comunes

## Criterios de Aceptación por Historia de Usuario

### HU1: Sincronizar Oferta Académica
**Dado que** el usuario inicia la app por primera vez
**Cuando** selecciona una carrera para sincronizar
**Entonces** debe ver:
- ✅ Lista de carreras cargada desde UMSS (mínimo 5 carreras válidas)
- ✅ Progreso visual durante sincronización (niveles → materias → grupos)
- ✅ Datos persistidos localmente tras sincronización exitosa
- ✅ Mensaje de confirmación y navegación automática a selección de niveles
- ✅ Reintento automático en caso de error de red (máximo 3 intentos)
- ✅ Estado offline preservado si falla conectividad

**Validación técnica**:
- ✅ Parsing PDF exitoso ≥90% de bloques horarios válidos
- ✅ Integridad referencial en base de datos (foreign keys)
- ✅ Transacciones rollback en caso de error parcial

### HU2: Seleccionar Materias y Grupos
**Dado que** el usuario tiene carreras sincronizadas
**Cuando** navega la jerarquía carrera→nivel→materia→grupo
**Entonces** debe poder:
- ✅ Ver jerarquía completa poblada con datos reales
- ✅ Seleccionar múltiples grupos por materia
- ✅ Ver preview de horarios con conflictos visuales
- ✅ Personalizar con colores, emojis y notificaciones
- ✅ Persistir selecciones tras navegación y reinicio app
- ✅ Validar restricciones básicas (horarios no superpuestos si requerido)

**Validación técnica**:
- ✅ Estado UI consistente con base de datos
- ✅ Flujos StateFlow reactivos sin memory leaks
- ✅ SavedStateHandle preserva selecciones en recreación

### HU3: Preferencias de Generación
**Dado que** el usuario tiene materias seleccionadas
**Cuando** configura criterios de optimización
**Entonces** debe poder:
- ✅ Ajustar pesos de estrategias (gaps, días libres, docentes)
- ✅ Marcar docentes como favoritos con toggle visual
- ✅ Ver impacto de cambios en preview de score
- ✅ Persistir configuraciones en UserSettings
- ✅ Reset a valores por defecto

**Validación técnica**:
- ✅ Estrategias aplican pesos correctamente en cálculo de score
- ✅ CompositeStrategy combina múltiples criterios
- ✅ UI refleja cambios en tiempo real

### HU4: Generar Horarios Candidatos
**Dado que** el usuario tiene configuración completa
**Cuando** ejecuta generación de horarios
**Entonces** debe obtener:
- ✅ Al menos 3 candidatos viables sin conflictos duros
- ✅ Horarios ordenados por score (mejor primero)
- ✅ Métricas visibles (gaps totales, días utilizados, docentes favoritos)
- ✅ Tiempo de generación < 3 segundos para casos típicos
- ✅ Opción de regenerar con parámetros modificados

**Validación técnica**:
- ✅ Backtracking explora espacio combinatorio completo
- ✅ Poda temprana evita combinaciones inválidas
- ✅ Score finito para todos los candidatos válidos

### HU5: Aplicar Horario
**Dado que** el usuario selecciona un candidato
**Cuando** confirma aplicación del horario
**Entonces** debe ver:
- ✅ Vista semanal poblada con horario aplicado
- ✅ Navegación a detalle de materias/eventos
- ✅ Persistencia tras reinicio app
- ✅ Notificaciones programadas para clases futuras
- ✅ Opción de modificar horario aplicado

**Validación técnica**:
- ✅ SelectedSubject actualizado con groupId correcto
- ✅ Relaciones many-to-many consistentes
- ✅ WorkManager programa notificaciones correctamente

### HU6: Exportar Horario
**Dado que** el usuario tiene horario aplicado
**Cuando** solicita exportación
**Entonces** debe obtener:
- ✅ Archivos PDF/Excel/Imagen/JSON generados sin errores
- ✅ Contenido completo y legible del horario semanal
- ✅ Nombre de archivo consistente (timestamp + formato)
- ✅ Opción de compartir vía intents del sistema
- ✅ Preview antes de generación

**Validación técnica**:
- ✅ Generadores especializados por formato
- ✅ Memoria eficiente para grandes horarios
- ✅ Encoding correcto para caracteres especiales

### HU7: Importar Horario Compartido
**Dado que** el usuario recibe JSON de horario
**Cuando** importa el archivo
**Entonces** debe poder:
- ✅ Validar esquema JSON contra estructura esperada
- ✅ Ver preview del horario a importar
- ✅ Elegir fusión vs reemplazo completo
- ✅ Aplicar sin corrupción de datos existentes
- ✅ Mensaje de éxito con estadísticas de importación

**Validación técnica**:
- ✅ Parsing JSON robusto con error handling
- ✅ Validación semántica de referencias
- ✅ Transacciones para atomicidad

### HU8: Notificaciones de Clases
**Dado que** el usuario tiene horario aplicado
**Cuando** se acerca la hora de clase
**Entonces** debe recibir:
- ✅ Notificación 5-15 minutos antes (configurable)
- ✅ Información completa (materia, aula, docente)
- ✅ Acciones: posponer 5 min, descartar
- ✅ Sonido/vibración según configuración
- ✅ Respeta Do Not Disturb

**Validación técnica**:
- ✅ WorkManager con constraints de batería
- ✅ Exact timing con AlarmManager para precisión
- ✅ PendingIntents seguros para acciones

### HU9: Eventos Académicos Manuales
**Dado que** el usuario quiere agregar evento personalizado
**Cuando** crea evento (examen, reunión)
**Entonces** debe poder:
- ✅ CRUD completo con validación de campos
- ✅ Visualización en calendario semanal
- ✅ Detección de conflictos con clases existentes
- ✅ Notificaciones programadas para eventos
- ✅ Categorización (examen, tarea, reunión)

**Validación técnica**:
- ✅ AcademicEventEntity con relaciones correctas
- ✅ Validación de solapamientos en lógica de dominio
- ✅ Cascade deletes para integridad

### HU10: Copia de Seguridad
**Dado que** el usuario quiere respaldar datos
**Cuando** ejecuta backup/restore
**Entonces** debe:
- ✅ Generar JSON completo de todos los datos usuario
- ✅ Subir a Firebase Storage de forma segura
- ✅ Restaurar desde backup con confirmación
- ✅ Verificar integridad post-restore
- ✅ Progreso visual para operaciones largas

**Validación técnica**:
- ✅ Serialización completa del grafo de objetos
- ✅ Encriptación de datos sensibles
- ✅ Validación de versiones para compatibilidad

## Criterios de Aceptación por Épica

### E1: Sincronización de Oferta Académica
- ✅ Endpoint discovery automático
- ✅ Parsing PDF robusto con fallbacks
- ✅ DAOs transaccionales con constraints
- ✅ UI jerárquica poblada dinámicamente

### E2: Selección y Preferencias
- ✅ Flujo jerárquico intuitivo
- ✅ Personalización visual completa
- ✅ Preferencias persistidas y aplicadas

### E3: Generación y Aplicación
- ✅ Algoritmo backtracking + heurísticas
- ✅ Score compuesto configurable
- ✅ Vista semanal interactiva

### E4: Exportación/Importación
- ✅ Múltiples formatos soportados
- ✅ Validación y preview
- ✅ Sharing nativo

### E5: Notificaciones y Eventos
- ✅ Scheduling preciso con WorkManager
- ✅ CRUD eventos con validación
- ✅ Acciones en notificaciones

### E6: Backup/Restore
- ✅ JSON schema versionado
- ✅ Firebase integration segura
- ✅ Restore con merge options

### E7: Calidad y Despliegue
- ✅ Testing pirámide completa
- ✅ CI/CD automatizado
- ✅ Release management

## Métricas de Validación

### Automatizadas (CI/CD)
- ✅ Tests unitarios: >70% cobertura
- ✅ Tests integración: flujos críticos pasan
- ✅ Build release: sin warnings
- ✅ Lint: cero errores críticos

### Manuales (QA)
- ✅ Exploratory testing: 0 bugs críticos
- ✅ Usability testing: >4.5/5 satisfacción
- ✅ Performance: operaciones <3s
- ✅ Accessibility: navegación completa por teclado

### Métricas de Producto
- ✅ Crash-free users: >99%
- ✅ Feature adoption: >80% usuarios usan generación
- ✅ Time to complete: flujos principales <5 min
- ✅ Error recovery: >95% casos manejados gracefully
