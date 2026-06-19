# Revisión de código - SimpsonsApp

Este informe detalla una selección de **10 errores y malas prácticas** identificados en el proyecto, que abarcan desde errores de sintaxis que impiden la compilación hasta fallos en la arquitectura y lógica de la aplicación.

**Nota sobre la metodología:** Se utilizó el agente de **GEMINI** para acelerar la búsqueda y detección de errores técnicos en el código fuente, mientras que la selección final de los casos presentados en este informe fue realizada por mi.

### 1. app/src/main/java/com/example/simpsonsapp/domain/model/Episode.kt (Línea 11)
**Lo que está mal:** El bloque `init` está declarado fuera de las llaves de la `data class`. Es un error de sintaxis que impide compilar.
**Lo que debería hacerse:** Eliminar el bloque o moverlo dentro de la clase.
```kotlin
// Fix rápido: Eliminar líneas 11 a 13.
```

### 2. app/src/main/java/com/example/simpsonsapp/domain/repository/EpisodeRepository.kt (Línea 7)
**Lo que está mal:** El método está definido como `get_episodes()` (snake_case), lo cual no sigue las convenciones de nomenclatura de Kotlin y genera un error de coincidencia con la implementación.
**Lo que debería hacerse:** Renombrar el método a `getEpisodes()` en la interfaz para cumplir con las convenciones de camelCase y corregir el error de compilación.
```kotlin
// Fix rápido (Línea 7): 
fun getEpisodes(): Flow<PagingData<Episode>>
```

### 3. app/src/main/java/com/example/simpsonsapp/di/DataModule.kt (Línea 33)
**Lo que está mal:** Configuración incompleta de Retrofit. Falta la llamada a `.baseUrl()`, lo que lanzará una excepción en tiempo de ejecución.
**Lo que debería hacerse:** Agregar la URL base correspondiente antes de buildear la instancia.
```kotlin
// Fix rápido (Línea 33): 
.baseUrl("https://thesimpsonsapi.com/api/")
```

### 4. app/src/main/java/com/example/simpsonsapp/data/remote/EpisodeRemoteMediator.kt (Línea 105)
**Lo que está mal:** Uso de URL absoluta en la anotación `@GET`. Esto ignora la configuración global de la API.
**Lo que debería hacerse:** Usar una ruta relativa (ej. `"episodes"`) delegando la URL base al módulo de red.
```kotlin
// Fix rápido (Línea 105): 
@GET("episodes")
```

### 5. app/src/main/java/com/example/simpsonsapp/main/MainScreen.kt (Línea 50)
**Lo que está mal:** Efecto secundario no controlado. `viewModel.refreshSeasons()` se llama directamente en el cuerpo del Composable, ejecutándose en cada recomposición.
**Lo que debería hacerse:** Envolver la llamada en un `LaunchedEffect` para controlar su ejecución.
```kotlin
// Fix rápido (Línea 50): 
LaunchedEffect(Unit) { viewModel.refreshSeasons() }
```

### 6. app/src/main/java/com/example/simpsonsapp/data/repository/EpisodeRepositoryImpl.kt (Línea 31)
**Lo que está mal:** Lógica de filtrado incompleta. `getEpisodesBySeason` solo usa la base de datos local y no implementa `RemoteMediator`.
**Lo que debería hacerse:** Integrar el mediador para que el filtrado por temporada también pueda obtener datos de la red si no están en caché.

### 7. app/src/main/java/com/example/simpsonsapp/AppNavigation.kt (Líneas 18 y 26)
**Lo que está mal:** Navegación basada en Strings literales ("main", "detail/..."). Aumenta el riesgo de errores y dificulta el refactor.
**Lo que debería hacerse:** Centralizar las rutas en una `sealed class` u objeto de constantes.

### 8. app/src/main/java/com/example/simpsonsapp/main/MainScreen.kt (Línea 87)
**Lo que está mal:** Uso de "Magic Number" `(1..25)` para filtrar episodios, asumiendo una cantidad fija por temporada.
**Lo que debería hacerse:** Obtener los datos de episodios disponibles de forma dinámica desde el repositorio.

### 9. app/src/main/java/com/example/simpsonsapp/data/remote/EpisodeRemoteMediator.kt (Línea 103)
**Lo que está mal:** Falta de cohesión. La interfaz `SimpsonsApi` está definida dentro del archivo del mediador.
**Lo que debería hacerse:** Extraer la interfaz a su propio archivo dentro del paquete `remote`.

### 10. app/src/main/java/com/example/simpsonsapp/data/DataRepository.kt y MainScreenViewModel.kt
**Lo que está mal:** Código muerto. El proyecto contiene archivos y clases que no se utilizan en ninguna parte de la aplicación real.
**Lo que debería hacerse:** Eliminar estos archivos para mejorar la mantenibilidad y limpieza del código.

---

**Nota:** Se han seleccionado exactamente 10 casos para este informe. Sin embargo, el proyecto presenta otras áreas de mejora posibles, tales como:
- Implementación de pruebas unitarias y de UI.
- Inyección de dependencias para los Dispatchers de Corrutinas.
- Mejora de la accesibilidad y soporte para diferentes tamaños de pantalla.
- Uso de un estado de UI unificado (UI State) en los ViewModels.
- Manejo avanzado de errores y estados de carga (Empty/Error states) con componentes de Material 3.
- Internacionalización de strings hardcodeados.
