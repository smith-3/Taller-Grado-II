# Sincronización Automática de Horarios

## Resumen

Se implementó una funcionalidad de sincronización automática que verifica actualizaciones en los horarios de carreras inscritas al iniciar la aplicación. Esta característica asegura que los usuarios siempre tengan la información más actualizada sin necesidad de intervención manual.

## Arquitectura Implementada

### Componentes Principales

#### 1. AutoSyncUseCase
- **Ubicación**: `domain/usecase/AutoSyncUseCase.kt`
- **Responsabilidad**: Lógica central de sincronización automática
- **Funciones**:
  - Obtiene carreras inscritas (`isEnabled = true`)
  - Compara `lastSyncTime` vs `updatedDate` del servidor
  - Ejecuta actualización si hay cambios
  - Actualiza `lastSyncTime` al completar

#### 2. SyncPreferences
- **Ubicación**: `data/local/preferences/SyncPreferences.kt`
- **Responsabilidad**: Persistencia del estado de sincronización
- **Funciones**:
  - Guarda timestamp del último check
  - Evita verificaciones innecesarias (< 6 horas)

#### 3. Modificaciones en HomeViewModel
- **Ubicación**: `presentation/home/HomeViewModel.kt`
- **Cambios**:
  - Añadido `checkForUpdates()` en `init` block
  - Integración con `AutoSyncUseCase` y `SyncPreferences`
  - Notificación push al completar actualización exitosa

#### 4. SyncDialog
- **Ubicación**: `presentation/home/SyncDialog.kt`
- **Responsabilidad**: Interfaz para informar sobre actualizaciones
- **Estado**: Placeholder preparado para futura implementación

### Flujo de Ejecución

```mermaid
graph TD
    A[App inicia] --> B[HomeViewModel.init]
    B --> C[checkForUpdates()]
    C --> D{Último check > 6h?}
    D -->|No| E[Continuar carga normal]
    D -->|Sí| F[Obtener carreras inscritas]
    F --> G[Comparar lastSyncTime vs updatedDate]
    G -->|Sin cambios| E
    G -->|Hay cambios| H[Ejecutar AutoSyncUseCase]
    H --> I[Actualizar BD]
    I --> J[Mostrar notificación push]
    J --> E
```

### Manejo de Errores

- **Sin internet**: Continúa carga normal sin mostrar errores
- **Error en scraper**: Log silencioso, continúa
- **Error en actualización**: Log por carrera, continúa con otras
- **Persistencia**: Actualiza timestamp solo en éxito

### Integración con Arquitectura Existente

- **Reutilización**: Aprovecha `RefreshSubjectsForCareerUseCase`, `ImportCareerSchedulesUseCase`
- **Inyección**: Hilt para dependencias
- **Coroutines**: IO dispatcher para operaciones de red/BD
- **MVVM**: Patrón existente mantenido

### Configuración

- **Intervalo**: 6 horas entre verificaciones
- **Alcance**: Solo carreras inscritas (`isEnabled = true`)
- **Notificación**: Push notification al completar actualización

### Beneficios

1. **Transparente**: Funciona en background sin interrumpir UX
2. **Eficiente**: Evita verificaciones innecesarias
3. **Robusto**: Manejo de errores silencioso
4. **Escalable**: Fácil añadir configuración futura (WorkManager, etc.)

### Próximas Mejoras

- **WorkManager**: Sincronización periódica en background
- **Configuración**: Opción de intervalo en settings
- **Cache**: Evitar re-descargas de PDFs
- **Analytics**: Métricas de uso

## Archivos Modificados/Creados

### Nuevos
- `src/main/java/com/example/tecnotime/domain/usecase/AutoSyncUseCase.kt`
- `src/main/java/com/example/tecnotime/data/local/preferences/SyncPreferences.kt`
- `src/main/java/com/example/tecnotime/presentation/home/SyncDialog.kt`
- `docs/sincronizacion-automatica.md`

### Modificados
- `src/main/java/com/example/tecnotime/di/AppModule.kt` (dependencias Hilt)
- `src/main/java/com/example/tecnotime/presentation/home/HomeViewModel.kt` (lógica de check)

## Testing

La implementación incluye manejo de errores y logging para facilitar debugging. Se recomienda probar:

1. Con internet disponible y actualizaciones pendientes
2. Sin internet
3. Con errores en el scraper
4. Múltiples carreras inscritas
5. Verificación del intervalo de 6 horas