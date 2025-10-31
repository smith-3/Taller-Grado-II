# Backlog y Tablero de Desarrollo

## Metodología de Gestión

### Kanban Adaptado
Se implementó un sistema Kanban simplificado para desarrollo individual:
- **Columnas**: Backlog → En Progreso → Testing → Hecho
- **WIP Limit**: Máximo 3 tareas en progreso simultáneo
- **Tareas**: Granulares, completables en 1-2 días
- **Priorización**: MoSCoW (Must have, Should have, Could have, Won't have)

### Estimación y Planning
- **Story Points**: Usando escala Fibonacci modificada (1,2,3,5,8)
- **Velocidad**: 8-12 puntos por semana (desarrollo individual)
- **Refinement**: Sesiones semanales para refinar backlog
- **Retrospectives**: Al final de cada épica para lecciones aprendidas

## Épicas y Historias de Usuario

### E1: Sincronización de Oferta Académica (HU1)
**Objetivo**: Obtener y mantener datos actualizados de UMSS
**Historias**: HU1 (Sincronizar oferta académica)
**Criterios de Épica**: Scraping funcional, datos persistidos, UI poblada

### E2: Selección y Preferencias de Generación (HU2, HU3)
**Objetivo**: Permitir selección personalizada de materias y criterios
**Historias**: HU2 (Seleccionar materias), HU3 (Preferencias generación)
**Criterios de Épica**: Flujo selección completo, preferencias persistidas

### E3: Generación y Aplicación de Horarios (HU4, HU5)
**Objetivo**: Algoritmo inteligente de generación de horarios
**Historias**: HU4 (Generar candidatos), HU5 (Aplicar horario)
**Criterios de Épica**: Generación <3s, resultados óptimos, aplicación correcta

### E4: Exportación/Importación (HU6, HU7)
**Objetivo**: Compartir y respaldar horarios
**Historias**: HU6 (Exportar), HU7 (Importar)
**Criterios de Épica**: Múltiples formatos, validación robusta

### E5: Notificaciones y Eventos (HU8, HU9)
**Objetivo**: Recordatorios y gestión de eventos
**Historias**: HU8 (Notificaciones clases), HU9 (Eventos manuales)
**Criterios de Épica**: Scheduling preciso, CRUD eventos funcional

### E6: Backup/Restore (HU10)
**Objetivo**: Persistencia de datos del usuario
**Historias**: HU10 (Copia de seguridad)
**Criterios de Épica**: Backup completo, restore íntegro

### E7: Calidad y Despliegue
**Objetivo**: Producto listo para producción
**Historias**: Testing, CI/CD, documentación
**Criterios de Épica**: Cobertura 70%+, pipeline automatizado, release exitoso

## Backlog Detallado

### En Progreso
- [ ] Implementar algoritmo backtracking con poda para generación de horarios
- [ ] Crear UI para selección jerárquica de materias y grupos
- [ ] Implementar sistema de notificaciones con WorkManager

### Hecho ✅
- [x] Configurar arquitectura Clean con Hilt y Room
- [x] Implementar scraping básico de PDFs UMSS
- [x] Crear modelos de dominio y entidades Room
- [x] Implementar DAOs con transacciones para integridad
- [x] Crear UI básica de navegación con Compose
- [x] Implementar repositorios y casos de uso iniciales

### Backlog Priorizado

#### Alta Prioridad (Must Have)
- [ ] Finalizar algoritmo de generación con estrategias compuestas
- [ ] Implementar exportación a PDF/Excel/Imagen
- [ ] Crear sistema de permisos y notificaciones
- [ ] Implementar backup/restore con Firebase
- [ ] Testing de integración para flujos críticos

#### Media Prioridad (Should Have)
- [ ] Optimizar performance de generación (memoización, paralelización)
- [ ] Añadir importación de horarios JSON
- [ ] Implementar gestión de eventos académicos
- [ ] Mejorar UX con animaciones y feedback
- [ ] Añadir métricas y logging avanzado

#### Baja Prioridad (Could Have)
- [ ] Implementar modo offline avanzado
- [ ] Añadir themes dinámicos
- [ ] Integrar analytics de uso
- [ ] Crear widget de escritorio
- [ ] Implementar compartir horarios vía deep links

## Riesgos y Estrategias de Mitigación

### Riesgos Técnicos
- **Cambio estructura UMSS**: Mitigación con regex configurables y tests de integración
- **Complejidad combinatoria**: Mitigación con timeouts y poda agresiva
- **Performance móvil**: Mitigación con profiling y optimizaciones
- **Batería/WorkManager**: Mitigación con constraints y batching

### Riesgos de Proyecto
- **Alcance creep**: Mitigación con definición clara de MVP
- **Dependencias externas**: Mitigación con fallbacks y offline-first
- **Cambios requisitos**: Mitigación con iteraciones cortas y feedback

### Riesgos de Calidad
- **Deuda técnica**: Mitigación con refactoring continuo
- **Bugs en producción**: Mitigación con testing automatizado
- **UX issues**: Mitigación con user testing y analytics

## Métricas de Progreso

### Burndown Chart (Estimado)
- **Semana 1-2**: Setup arquitectura y modelos (15 puntos)
- **Semana 3-4**: Scraping y sincronización (20 puntos)
- **Semana 5-6**: UI selección y preferencias (18 puntos)
- **Semana 7-8**: Algoritmo generación (25 puntos)
- **Semana 9-10**: Export/import y notificaciones (20 puntos)
- **Semana 11-12**: Testing y despliegue (15 puntos)

### Hitos de Entrega
- **H1 (Fin Semana 4)**: Scraping funcional, datos en Room, UI básica
- **H2 (Fin Semana 8)**: Generación operacional, flujos principales completos
- **H3 (Fin Semana 10)**: Features completas, testing básico aprobado
- **H4 (Fin Semana 12)**: Release candidate, documentación finalizada

## Lecciones Aprendidas

### Iteración 1: Arquitectura
- **Aprendido**: Clean Architecture escala bien para móvil
- **Mejora**: Más inversión inicial en abstracciones paga dividends
- **Aplicado**: Patrón Strategy para algoritmos intercambiables

### Iteración 2: Scraping
- **Aprendido**: PDFs son frágiles, regex necesita configuración externa
- **Mejora**: Tests de integración desde temprano
- **Aplicado**: Validación robusta y logging agresivo

### Iteración 3: Algoritmos
- **Aprendido**: Backtracking + heurísticas es powerful pero complejo
- **Mejora**: Prototipado rápido antes de optimización
- **Aplicado**: Estrategias compuestas con pesos configurables

## Próximas Acciones
1. Completar algoritmo de generación con testing
2. Implementar exportación PDF con iText
3. Crear sistema notificaciones con permisos
4. Refactorizar UI para mejor UX
5. Implementar CI/CD básico con GitHub Actions
