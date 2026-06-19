2do Parcial - Parte Practica
Que se solicita:

El codigo tiene 10 errores. Recae en usted analizar que es un error dentro del codigo.
Los Alumnos tendran que forkear este repo como propio, hacer un issue desde Github con Comentarios refiriendo en que linea esta el error, y como se debe solucionar.
La respuesta sera con el link a ese Fork, y adentro deben estar los issues. Los profesores tenemos que poder ingresar al mismo. Recae en los alumnos asegurarse de que los profesores puedan ingresar.
Tambien pueden editar el Archivo Readme y poner los resultados dentro de sus propios forks.
https://github.com/ExBattou/SimpsonsApp

---

## Errores encontrados

### Error 1 — `Episode.kt`, líneas 12-14
**Bloque `init` suelto fuera de la clase**

```kotlin
// INCORRECTO
init {
    return Episode; //NO BORRAR
}
```

Hay un bloque `init { return Episode; }` escrito fuera del cuerpo de la data class `Episode`. En Kotlin, un bloque `init` solo puede existir dentro de una clase, y además no puede tener un `return` con valor. Este código no compila.

**Solución:** eliminar completamente ese bloque, ya que es inválido y no tiene ningún propósito funcional.

---

### Error 2 — `EpisodeRepository.kt`, línea 8
**Nombre de método incorrecto: `get_episodes()` en lugar de `getEpisodes()`**

```kotlin
// INCORRECTO
fun get_episodes(): Flow<PagingData<Episode>>
```

La interfaz `EpisodeRepository` declara el método como `get_episodes()` (con guion bajo), pero la implementación en `EpisodeRepositoryImpl` lo implementa como `getEpisodes()` (camelCase). Esto rompe el contrato de la interfaz y no compila.

**Solución:** renombrar la declaración en la interfaz a `getEpisodes()`.

```kotlin
// CORRECTO
fun getEpisodes(): Flow<PagingData<Episode>>
```

---

### Error 3 — `EpisodeRepositoryImpl.kt`, línea 18
**Importación faltante de `SimpsonsApi`**

```kotlin
// INCORRECTO — SimpsonsApi no está importado
class EpisodeRepositoryImpl @Inject constructor(
    private val simpsonsApi: SimpsonsApi,
    ...
```

`EpisodeRepositoryImpl` usa `SimpsonsApi` como dependencia, pero `SimpsonsApi` está definida en el paquete `data.remote` y no hay ningún import de ese paquete en el archivo. El código no compilará.

**Solución:** agregar el import correspondiente.

```kotlin
import com.example.simpsonsapp.data.remote.SimpsonsApi
```

---

### Error 4 — `EpisodeRemoteMediator.kt`, líneas 13-14
**Imports incorrectos: `retrofit2.http.GET` y `retrofit2.http.Query` dentro de un archivo que no define endpoints HTTP directamente**

```kotlin
import retrofit2.http.GET
import retrofit2.http.Query
```

Estos imports corresponden a las anotaciones de Retrofit que se usan en la interfaz `SimpsonsApi`, la cual sí está definida en el mismo archivo. Sin embargo, la clase `EpisodeRemoteMediator` en sí misma no usa esas anotaciones. Si la interfaz `SimpsonsApi` fuera movida a su propio archivo (como es la práctica correcta), estos imports quedarían huérfanos y el mediator quedaría acoplado innecesariamente. La interfaz `SimpsonsApi` debería estar en su propio archivo separado.

**Solución:** mover la interfaz `SimpsonsApi` a un archivo propio `SimpsonsApi.kt` dentro del paquete `data.remote`, y eliminar los imports de `retrofit2.http.GET` y `retrofit2.http.Query` del mediator.

---

### Error 5 — `EpisodeRemoteMediator.kt` / `SimpsonsApi`, línea con `@GET`
**URL absoluta en la anotación `@GET` en lugar de una URL relativa**

```kotlin
// INCORRECTO
@GET("https://thesimpsonsapi.com/api/episodes")
suspend fun getEpisodes(@Query("page") page: Int): EpisodesResponse
```

Retrofit espera que las anotaciones `@GET` reciban una ruta relativa al `baseUrl` configurado en el builder. Usar una URL absoluta directamente en `@GET` es una mala práctica y puede fallar dependiendo de la configuración del cliente.

**Solución:** definir el `baseUrl` en el `Retrofit.Builder` dentro de `DataModule.kt` y usar solo la ruta relativa en la anotación.

```kotlin
// En DataModule.kt
.baseUrl("https://thesimpsonsapi.com/")

// En SimpsonsApi
@GET("api/episodes")
suspend fun getEpisodes(@Query("page") page: Int): EpisodesResponse
```

---

### Error 6 — `DataModule.kt`, líneas 33-37
**Falta el `baseUrl` en el builder de Retrofit**

```kotlin
// INCORRECTO
return Retrofit.Builder()
    .client(client)
    .addConverterFactory(GsonConverterFactory.create())
    .build()
    .create(SimpsonsApi::class.java)
```

Retrofit requiere obligatoriamente que se llame a `.baseUrl(...)` antes de `.build()`. Sin eso, Retrofit lanza una `IllegalStateException` en tiempo de ejecución.

**Solución:** agregar el `baseUrl`.

```kotlin
return Retrofit.Builder()
    .baseUrl("https://thesimpsonsapi.com/")
    .client(client)
    .addConverterFactory(GsonConverterFactory.create())
    .build()
    .create(SimpsonsApi::class.java)
```

---

### Error 7 — `AppNavigation.kt`, líneas 9-10
**Imports con wildcard innecesarios e incorrectos**

```kotlin
import androidx.navigation.*
import androidx.compose.*
```

El import `androidx.compose.*` no existe como paquete válido (el paquete raíz de Compose no exporta nada directamente), y causa un error de compilación. Además, usar wildcards en lugar de imports específicos es una mala práctica.

**Solución:** eliminar ambas líneas de wildcard. Todos los símbolos necesarios ya están importados explícitamente en las líneas anteriores del mismo archivo.

---

### Error 8 — `MainScreen.kt`, línea con `LazyRow`
**Se usa `LazyRow` (scroll horizontal) para listar episodios en lugar de `LazyColumn` (scroll vertical)**

```kotlin
// INCORRECTO
LazyRow(
    state = listState,
    modifier = Modifier.fillMaxSize()
) { ... }
```

Una lista de episodios es contenido que naturalmente se navega de forma vertical. Usar `LazyRow` hace que los cards se desplacen horizontalmente, lo cual es una experiencia de usuario incorrecta para este caso. El indicador de carga horizontal con `fillMaxHeight()` también es inapropiado para un layout vertical.

**Solución:** reemplazar `LazyRow` por `LazyColumn` y ajustar el indicador de carga a un ancho completo.

---

### Error 9 — `EpisodeEntity.kt`, línea 7
**Se usa `class` en lugar de `data class` para la entidad de Room**

```kotlin
// INCORRECTO
class EpisodeEntity(
    @PrimaryKey val id: Int,
    ...
)
```

Room genera código basado en las propiedades de la entidad. Aunque técnicamente Room puede trabajar con clases normales, no tener `data class` significa que no se generan automáticamente `equals()`, `hashCode()` ni `toString()`, lo que puede causar bugs sutiles en comparaciones de objetos y en el funcionamiento del `DiffCallback` de Paging.

**Solución:** declarar `EpisodeEntity` como `data class`.

```kotlin
// CORRECTO
data class EpisodeEntity(
    @PrimaryKey val id: Int,
    ...
)
```

---

### Error 10 — `RemoteKeyEntity.kt`, línea 7
**Se usa `class` en lugar de `data class` para la entidad de Room**

```kotlin
// INCORRECTO
class RemoteKeyEntity(
    @PrimaryKey val episodeId: Int,
    ...
)
```

Mismo problema que `EpisodeEntity`: al no ser `data class`, se pierden las implementaciones automáticas de `equals()` y `hashCode()`, lo que puede afectar la lógica de paginación remota.

**Solución:** declarar `RemoteKeyEntity` como `data class`.

```kotlin
// CORRECTO
data class RemoteKeyEntity(
    @PrimaryKey val episodeId: Int,
    ...
)
```

---

## Bonus — `gradle-wrapper.properties`
**Versión de Gradle incompatible con el Android Gradle Plugin**

```properties
# INCORRECTO
distributionUrl=https\://services.gradle.org/distributions/gradle-9.1.0-bin.zip
```

El proyecto usaba Gradle `9.1.0`, pero el Android Gradle Plugin configurado es `8.6.0`, que requiere **Gradle 8.7 como máximo**. Esta incompatibilidad impide que el proyecto buildee completamente.

**Solución:** bajar la versión de Gradle a `8.7` en `gradle-wrapper.properties`.

```properties
# CORRECTO
distributionUrl=https\://services.gradle.org/distributions/gradle-8.7-bin.zip
distributionSha256Sum=194717442575a6f96e1c1befa2c30e9a4fc90f701d7aee353a71b0f290189bc8
```

---

## Bonus 2 — `gradle.properties`
**Ruta de JDK hardcodeada para macOS, incompatible con Windows**

```properties
# INCORRECTO
org.gradle.java.home=/opt/homebrew/Cellar/openjdk@17/17.0.15/libexec/openjdk.jdk/Contents/Home
```

La propiedad `org.gradle.java.home` estaba apuntando a una ruta de macOS (Homebrew), lo que hace que el build falle en Windows con el error `Java home supplied is invalid`.

**Solución:** reemplazar la ruta por el JDK bundleado de Android Studio en Windows, o eliminar la línea para que Gradle use el JDK del sistema.

```properties
# CORRECTO (Windows con Android Studio)
org.gradle.java.home=C:\\Program Files\\Android\\Android Studio\\jbr
```
