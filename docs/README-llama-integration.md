# Integración de `llama.cpp` en TecnoTime

> Guía práctica para mantener, depurar y extender el motor de IA local dentro de la aplicación Android.

---

## 1. Visión general

La app integra un modelo GGUF vía `llama.cpp`, empaquetado como un módulo Android (`llama`). La pila tiene tres capas:

1. **Wrapper nativo (`LLamaAndroid`)**  
   Gestiona un hilo dedicado, la carga del modelo, el contexto y expone `send()` como `Flow<String>`.
2. **Servicios Kotlin (`AiService`)**  
   Orquesta el wrapper, asegura configuración, concatena tokens, aplica timeouts y errores amigables.
3. **Casos de uso / UI (`GenerateTextUseCase`, `GenerateAiNotificationUseCase`, `AiManagementViewModel`)**  
   Verifican ajustes del usuario, lanzan descargas, generan notificaciones y actualizan `uiState`.

```
Compose UI ──▶ ViewModel ──▶ Use Case ──▶ AiService ──▶ LLamaAndroid ──▶ llama-android.cpp ──▶ llama.cpp
```

---

## 2. Requisitos previos

- **Hardware**: dispositivo/emulador con ≥6 GB RAM para modelos de 2–3 GB (OLMoE por defecto).
- **Archivos**: `.gguf` descargado en `context.getExternalFilesDir(null)` (lo maneja `ModelDownloader`).
- **Permisos**: almacenamiento y notificaciones habilitadas.
- **Dependencias**: módulo `llama` compila contra una copia local de `llama.cpp` (ver `llama/src/main/cpp/CMakeLists.txt`).

---

## 3. Flujo de inicialización

1. `AiManagementViewModel` ejecuta `loadCurrentSettings()` y `checkExistingDownloads()`.
2. `ModelDownloader` determina si hay descargas en curso o archivo existente.
3. Al confirmar la descarga, se monitoriza `ModelDownloader.DownloadState` para actualizar la UI.
4. `generateTestNotification()` llama `modelInitializationService.getDefaultModel()` y, si es necesario, `aiService.ensureModelLoaded(path)`.
5. Una vez cargado, el caso de uso ejecuta `GenerateTextUseCase.invokeOneShot(prompt, config)` utilizando un `AiGenerationConfig` con el presupuesto de tokens adecuado para la experiencia.

---

## 4. Wrapper nativo (`LLamaAndroid.kt`)

Archivo: `llama/src/main/java/android/llama/cpp/LLamaAndroid.kt`

- Crea un `SingleThreadExecutor` (`Llm-RunLoop`) donde se inicializa la librería nativa.
- `load(path)` hace:
  - `load_model` → modelo
  - `new_context` → contexto
  - `new_batch(512, embd=0, seq=1)`
  - `new_sampler()` → sampler configurado en C++
- El runtime expone `setRuntimeConfig(...)` para ajustar `temperature`, `topP`, `topK`, `repeatPenalty`, `minP`, `typicalP`, `maxOutputTokens` y el límite de tokens del contexto sin recompilar la librería.
- `send(message, maxOutputTokens)` devuelve un `Flow<String>`; cada iteración:
  - Llama `completion_init()` (tokeniza prompt y decodifica una vez) solicitando sólo los tokens adicionales necesarios.
  - Calcula el límite `promptTokens + maxOutputTokens` (respetando `n_ctx=2048`) y repite `completion_loop()` hasta alcanzarlo o hasta recibir `null`.
  - Emite cada chunk al flujo Kotlin y limpia el KV cache al final.
- **Logging**: `Log.v/tag LLamaAndroid` reporta tokens iniciales, chunk emitido, longitud final.

---

## 5. Servicio de IA (`AiService.kt`)

Archivo: `src/main/java/com/example/tecnotime/domain/ai/service/AiService.kt`

- `ensureModelLoaded(path)` verifica que la IA esté activada y carga el modelo una sola vez con un `Mutex`.
- `generateTextStream(prompt, config)` expone el `Flow<String>` de `LLamaAndroid` con control de tokens y aplica el `LlamaRuntimeConfig` antes de enviar el prompt.
- `generateTextOnce(prompt, minChars, config)`:
  - Ejecuta en `Dispatchers.Default`
  - Aplica un timeout configurable: respeta `config.timeoutMillis` y, si no se define, deriva uno dinámico según los tokens permitidos con un mínimo de ~60s por seguridad
  - Registra cada chunk (`chunk[idx]`, longitud, contenido visible)
  - Concatena tokens y recorta `trim()`
  - Lanza `IllegalStateException("Model returned empty/short text")` si el resultado no alcanza `minChars`
- `benchSafe(...)` permite monitorear el rendimiento (`tokens/s`).

---

## 6. Casos de uso y UI

- **`GenerateTextUseCase`**  
  - Método streaming (`operator fun invoke`) → útil para chats; recibe `AiGenerationConfig`.  
  - Método one-shot (`invokeOneShot`) → agrega logs, valida ajustes, delega en `AiService` respetando el presupuesto de tokens.

- **`GenerateAiNotificationUseCase`**  
  - Confirma que `enableAi` esté activo.  
  - Llama `invokeOneShot(prompt, config)` con longitud mínima 30 y un `AiGenerationConfig` corto (p. ej. 60–80 tokens) para mensajes breves.  
  - Construye `NotificationInfo` y ejecuta `ShowNotificationUseCase`.

- **`AiManagementViewModel`**  
  - Sincroniza `UserSettings`.  
  - Gestiona descargas, diálogos de red y errores (`generationError`).  
  - Exponen `generateTestNotification()` para validar el pipeline end-to-end.

---

## 6.1 Configuración en tiempo de ejecución

- `AiGenerationConfig` controla el presupuesto de salida por feature (tokens adicionales, longitud mínima).
- `LlamaRuntimeConfig` agrupa parámetros de muestreo y ejecución (`temperature`, `topP`, `topK`, `repeatPenalty`, `maxContextTokens`).  
- `LlamaRuntimeConfigProvider` provee la configuración base (hoy in-memory, listo para persistir).  
- `AiService` combina ambos (`runtime.copy(maxOutputTokens = generationConfig.maxOutputTokens)`) y llama `IAiClient.updateRuntimeConfig(...)` antes de cada generación.
- El puente nativo recrea el sampler con los nuevos parámetros y respeta el límite de contexto definido.

---

## 7. Configuración del sampler (C++)

Archivo: `llama/src/main/cpp/llama-android.cpp`

- `Java_android_llama_cpp_LLamaAndroid_new_1sampler` crea la cadena de samplers inicial.
- `Java_android_llama_cpp_LLamaAndroid_recreate_1sampler` recibe los parámetros desde Kotlin y reconstruye la cadena en caliente:

  ```cpp
  llama_sampler_chain_add(chain, llama_sampler_init_penalties(64, repeat_penalty, 0, 0));
  if (top_k > 0) llama_sampler_chain_add(chain, llama_sampler_init_top_k(top_k));
  if (top_p < 1.0f) llama_sampler_chain_add(chain, llama_sampler_init_top_p(top_p, 1));
  if (min_p > 0.0f) llama_sampler_chain_add(chain, llama_sampler_init_min_p(min_p, 1));
  if (typical_p < 1.0f) llama_sampler_chain_add(chain, llama_sampler_init_typical(typical_p, 1));
  llama_sampler_chain_add(chain, llama_sampler_init_temp(temperature));
  llama_sampler_chain_add(chain, llama_sampler_init_dist(seed));
  ```

- `seed` se deriva de `System.nanoTime()` (vía Kotlin) para trazabilidad en logs.
- Ajusta estos parámetros mediante `LlamaRuntimeConfigProvider` sin recompilar; sólo necesitas volver a ejecutar `setRuntimeConfig(...)`.
- Si cambias la implementación nativa (por ejemplo para soportar GPU/offload), vuelve a ejecutar `./gradlew :app:assembleDebug` para generar los binarios actualizados.

---

## 8. Logging y depuración

- **Kotlin**
  - `LLamaAndroid`: prompt tokens, cada chunk (`chunk[idx] len=n -> 'contenido'`) y cierre del flujo.
  - `AiService`: modelLoaded, acumulado (`accumulatedLength`), `rawOutput`, errores por longitud.
  - `GenerateTextUseCase`: logs de inicio/fin, prompt, longitud objetivo.

- **Nativo (`llama-android.cpp`)**
  - Tokens originales del prompt (`token: \`...\` -> id`).
  - Avisos cuando `n_kv_req > n_ctx`.
  - Mensajes de sampler (`Initializing sampler (seed=...)`) y chunks sobre el búfer `cached_token_chars`.

- **Logcat sugeridos**  
  - Filtra por tags: `LLamaAndroid`, `AiService`, `GenerateTextUseCase`, `GenerateAiNotificationUseCase`, `ModelDownloader`.

---

## 9. Escenarios comunes y troubleshooting

| Problema                                   | Causa probable                                   | Solución                                                                 |
|-------------------------------------------|--------------------------------------------------|--------------------------------------------------------------------------|
| `Model not loaded`                        | `ensureModelLoaded` no se ejecutó o falló        | Invoca `AiService.ensureModelLoaded(path)` antes de generar.             |
| `Model returned empty/short text`         | El sampler termina en `<|endoftext|>` o timeout  | Revisa logs de chunks, sube `nLen`, ajusta sampling o prompt.            |
| `load_model() failed`                     | Ruta incorrecta o almacenamiento insuficiente    | Verifica `extFilesDir`, permisos y espacio disponible.                   |
| `n_kv_req > n_ctx`                        | Prompt muy largo para `n_ctx=2048`               | Reduce prompt o recompila con mayor contexto (costo de RAM).             |
| ANR/dispositivo se queda sin memoria       | Modelo grande en hardware limitado               | Usa modelos más pequeños (`Phi-2`, `TinyLlama`) o descarga opcional.     |
| Tokens repetitivos                         | Parámetros de sampler muy conservadores          | Ajusta `penalties`, `top_k`, `temperature` según el caso de uso.         |
| Descarga interrumpida                      | La app se cerró o cambió de red                  | `ModelDownloader` reanudará usando `getExistingDownload()` y monitoring. |

---

## 10. Extensiones y buenas prácticas

- **UI**: mostrar progreso de carga, opción de recargar modelo y mensajes claros ante errores.
- **Persistencia**: guarda prompts/respuestas relevantes en Room si necesitas auditoría.
- **Analítica**: mide duración, longitud de prompts, éxito/fallo y actualiza heurísticas.
- **Múltiples modelos**: amplía `getAvailableModels()` e implementa selector en configuración.
- **Fallback remoto**: ofrece un endpoint (Gemini/OpenAI) cuando el modelo local falla persistentemente.
- **Pruebas**: crea tests instrumentados que simulen la carga del modelo y generación sobre prompts cortos (ver `LlmWrapper`).
- **Seguridad**: evita almacenar prompts sensibles y ofrece controles en UI para desactivar la IA.

---

## 11. Flujo recomendado para nuevas integraciones

1. **Diseña el prompt** alineado con el tono y límite de caracteres del feature.
2. **Verifica ajustes de usuario** (`enableAi`, toggles específicos del feature).
3. **Asegura el modelo** (`ensureModelLoaded`), maneja carga o descarga previa.
4. **Configura el presupuesto de tokens** con `AiGenerationConfig` según el caso (notificaciones cortas, chat largo, etc.).
5. **Elige estrategia de output**  
   - `generateTextStream(config)` para chat o UI incremental.  
   - `generateTextOnce(config)` para respuestas breves (notificaciones, banners).
6. **Aplica postprocesamiento** (recorte, validación, sanitización).
7. **Registra métricas** para iterar: tiempos, longitud, errores.
8. **Documenta** el nuevo punto de entrada en esta guía para mantener coherencia.

---

## 12. Comandos útiles

```bash
# Compilar con binarios nativos actualizados
./gradlew :app:assembleDebug

# Solo Kotlin (sin recompilar C++)
./gradlew :app:compileDebugKotlin

# Benchmark rápido del modelo cargado (desde AiService)
aiService.benchSafe(pp = 512, tg = 128, pl = 1)
```

---

## 13. Recursos adicionales

- [Repositorio oficial de llama.cpp](https://github.com/ggerganov/llama.cpp)
- [Documentación de GGUF](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)
- [Android NDK Guides](https://developer.android.com/ndk)

Mantén esta guía actualizada cada vez que cambies prompts, sampling o nuevas áreas de negocio que dependan de la IA local. Esto asegura que el equipo tenga una fuente de verdad única para operar y depurar la integración con `llama.cpp`.***
