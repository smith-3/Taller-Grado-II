# AI Notification Customization (Roadmap)

Este documento resume las líneas de trabajo planeadas para enriquecer las notificaciones generadas por IA. Sirve como guía rápida antes de implementar los cambios.

## Objetivos
- Ofrecer estilos visuales diferenciados por categoría (motivación, recordatorios, humor, etc.).
- Mostrar el mensaje completo sin recortes, incluso en vista compacta.
- Incorporar elementos dinámicos: emojis, iconografía específica y acciones contextualizadas.
- Recibir feedback del usuario (p. ej., “me gustó/no me gustó”) para adaptar futuras notificaciones.

## Próximos pasos
1. **Metadatos generados por IA**  
   - Ajustar el prompt para solicitar categoría, emoji, color sugerido y etiquetas.
   - Definir un modelo `AiNotificationMetadata` que parse estos metadatos de la respuesta.

2. **Catálogo de estilos**  
   - Crear un catálogo local (`AiNotificationTheme`) que asigne iconos, colores y nivel de prioridad por categoría.
   - Permitir overrides manuales en ajustes para usuarios que prefieran estilos específicos.

3. **Constructor dinámico de notificaciones**  
   - Implementar un `AiNotificationStyler` que combine `NotificationInfo` + metadatos + tema seleccionado.
   - Soportar `BigTextStyle` y `BigPictureStyle`; usar `setLargeIcon()` o imágenes remotas cuando estén disponibles.

4. **Acciones inteligentes**  
   - Agregar botones para feedback (“❤️”, “👎”), “No molestar” o “Compartir”, usando `PendingIntent`s específicos.
   - Guardar la preferencia en `UserSettingsRepository` para personalizar envíos futuros.

5. **Layouts personalizados (opcional)**  
   - Diseñar un `RemoteViews` para visualizaciones avanzadas (chips de categoría, puntuación con corazones).
   - Incluir fallback a estilos estándar cuando el dispositivo o versión de Android no soporte el layout.

6. **Testing y degradación**  
   - Tests unitarios para el mapping de metadatos → estilo.
   - Validar en APIs clave (26, 30, 34) que el contenido largo se renderiza correctamente.
   - Definir un flujo de emergencia cuando falten metadatos (usar tema por defecto y acciones básicas).

## Estado actual
- Las notificaciones se muestran con `NotificationCompat.BigTextStyle`, asegurando que el texto completo sea visible.
- Falta integrar metadatos de IA, acciones de feedback y layouts personalizados.

## Recursos
- [Guía oficial de notificaciones](https://developer.android.com/develop/ui/views/notifications)
- [Custom notifications con RemoteViews](https://developer.android.com/develop/ui/views/notifications/custom-notifications)
- Revisar `docs/notificaciones.md` para detalles de la implementación vigente.
