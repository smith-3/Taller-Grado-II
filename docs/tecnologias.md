# Tecnologías Utilizadas en TecnoTime

## Lenguaje de Programación
- **Kotlin**: Lenguaje principal utilizado para el desarrollo de la aplicación Android, aprovechando sus características modernas como null safety y coroutines.

## Framework de Interfaz de Usuario
- **Jetpack Compose**: Framework declarativo para construir interfaces de usuario nativas en Android, utilizado para todas las pantallas y componentes de la aplicación.

## Arquitectura
- **MVVM (Model-View-ViewModel)**: Patrón de arquitectura para separar la lógica de presentación de la lógica de negocio.
- **Clean Architecture**: Principios de arquitectura limpia con capas bien definidas (Presentación, Dominio, Datos).

## Inyección de Dependencias
- **Hilt**: Framework de inyección de dependencias para Android, utilizado para gestionar las dependencias entre componentes.

## Base de Datos Local
- **Room**: Biblioteca de persistencia que proporciona una capa de abstracción sobre SQLite, utilizada para almacenar datos locales como horarios, materias y configuraciones.

## Sincronización y Autenticación Remota
- **Firebase Authentication**: Para autenticación de usuarios.
- **Firebase Firestore**: Base de datos NoSQL en la nube para sincronización de datos entre dispositivos.

## Scraping Web
- **Jsoup**: Biblioteca para parseo y manipulación de HTML, utilizada para extraer datos de horarios desde el sitio web de la UMSS.

## Generación de Documentos
- **PDFBox**: Para generación de archivos PDF con los horarios.
- **JXL (Java Excel API)**: Para exportación de horarios a formato Excel (.xls).
- **Jetpack Compose**: Utilizado para renderizar y generar imágenes de horarios semanales.

## Manejo de Fechas y Horas
- **ThreeTenABP**: Backport de java.time para versiones anteriores de Android, utilizado para manipulación de fechas y horas.

## Serialización JSON
- **Gson**: Biblioteca para conversión entre objetos Java/Kotlin y JSON, utilizada en importación/exportación de datos.

## Carga de Imágenes
- **Coil**: Biblioteca para carga y cacheo de imágenes, utilizada para mostrar logos y avatares.

## Navegación
- **Navigation Compose**: Componente de Jetpack para manejo de navegación entre pantallas en aplicaciones Compose.

## Notificaciones y Tareas en Segundo Plano
- **WorkManager**: Para programar tareas en segundo plano, como notificaciones de recordatorios de clases.

## Permisos y Utilidades de Compose
- **Accompanist Permissions**: Para manejo de permisos en aplicaciones Compose.
- **Accompanist System UI Controller**: Para control del sistema UI (barra de estado, navegación).
- **Accompanist Pager**: Para componentes de paginación.

## Soporte de Emojis
- **Emoji2**: Compatibilidad con emojis modernos en Android.

## Otras Dependencias
- **AndroidX Libraries**: Conjunto de bibliotecas modernas para Android (Core KTX, Lifecycle, Activity Compose, etc.).
- **Material Design 3**: Para componentes de UI consistentes con Material Design.
- **DataStore Preferences**: Para almacenamiento de configuraciones simples del usuario.
- **HTML to PDF Converter**: Para conversión adicional de HTML a PDF si es necesario.

## Herramientas de Desarrollo
- **Android Studio**: IDE oficial para desarrollo Android.
- **Gradle**: Sistema de build utilizado con Kotlin DSL (build.gradle.kts).
- **KSP (Kotlin Symbol Processing)**: Para procesamiento de anotaciones en tiempo de compilación (usado por Room y Hilt).

Esta combinación de tecnologías permite una aplicación robusta, escalable y con una experiencia de usuario moderna en Android.