# kotlinx.coroutines

Using in your projects
Maven
Add dependencies (you can also add other modules that you need):
```
<dependency>
    <groupId>org.jetbrains.kotlinx</groupId>
    <artifactId>kotlinx-coroutines-core</artifactId>
    <version>1.10.1</version>
</dependency>
```
And make sure that you use the latest Kotlin version:
```
<properties>
    <kotlin.version>2.0.0</kotlin.version>
</properties>
```
Gradle
Add dependencies (you can also add other modules that you need):
```
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.10.1")
}
```
And make sure that you use the latest Kotlin version:
```
plugins {
    // For build.gradle.kts (Kotlin DSL)
    kotlin("jvm") version "2.0.0"
    
    // For build.gradle (Groovy DSL)
    id "org.jetbrains.kotlin.jvm" version "2.0.0"
}
```
Make sure that you have mavenCentral() in the list of repositories:
```
repositories {
    mavenCentral()
}
```
Android
Add kotlinx-coroutines-android module as a dependency when using kotlinx.coroutines on Android:
```
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.10.1")
```
This gives you access to the Android Dispatchers.Main coroutine dispatcher and also makes sure that in case of a crashed coroutine with an unhandled exception that this exception is logged before crashing the Android application, similarly to the way uncaught exceptions in threads are handled by the Android runtime.

R8 and ProGuard
R8 and ProGuard rules are bundled into the kotlinx-coroutines-android module. For more details see "Optimization" section for Android.

Avoiding including the debug infrastructure in the resulting APK
The kotlinx-coroutines-core artifact contains a resource file that is not required for the coroutines to operate normally and is only used by the debugger. To exclude it at no loss of functionality, add the following snippet to the android block in your Gradle file for the application subproject:
```
packagingOptions {
    resources.excludes += "DebugProbesKt.bin"
}
```
Multiplatform
Core modules of kotlinx.coroutines are also available for Kotlin/JS and Kotlin/Native.

In common code that should get compiled for different platforms, you can add a dependency to kotlinx-coroutines-core right to the commonMain source set:
```
commonMain {
    dependencies {
        implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.10.1")
    }
}
```
Platform-specific dependencies are recommended to be used only for non-multiplatform projects that are compiled only for target platform.

JS
Kotlin/JS version of kotlinx.coroutines is published as kotlinx-coroutines-core-js (follow the link to get the dependency declaration snippet).

Native
Kotlin/Native version of kotlinx.coroutines is published as kotlinx-coroutines-core-$platform where $platform is the target Kotlin/Native platform. Targets are provided in accordance with official K/N target support.

Building and Contributing
See Contributing Guidelines.



```
dependencies {

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.ui.test.junit4)
    debugImplementation(libs.androidx.ui.tooling)
    debugImplementation(libs.androidx.ui.test.manifest)

    implementation(libs.androidx.lifecycle.viewmodel.compose)

    implementation(libs.kotlinx.coroutines.core)

    implementation(libs.coil.compose)
    implementation(libs.coil.network.okhttp)


    implementation(libs.ktor.serialization.kotlinx.json)
    implementation(libs.ktor.client.content.negotiation)
    implementation(libs.ktor.client.core)
    implementation(libs.ktor.client.cio)
    implementation (libs.ktor.client.android)


}
```

```
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    kotlin("plugin.serialization")
}
```

```
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    kotlin("plugin.serialization") version "2.1.20" apply false
}
```

AnimeResponse
```
import kotlinx.serialization.Serializable

@Serializable
data class AnimeResponse(
    val results: List<AnimeResult>
)

@Serializable
data class AnimeResult(
    val url: String,
    val artist_name: String? = null,
    val source_url: String? = null
)
```


AnimeRepository
```
import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
import io.ktor.client.request.get
import io.ktor.serialization.kotlinx.json.json
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import kotlinx.serialization.json.Json

class AnimeRepository {
    private val client = HttpClient {
        install(ContentNegotiation) {
            json(Json {
                ignoreUnknownKeys = true
                explicitNulls = false
            })
        }
    }

    suspend fun getAnime(category: String, count: Int): List<String> {
        val response = client.get("https://nekos.best/api/v2/$category?amount=$count")
            .body<AnimeResponse>()
        return response.results.map { it.url }
    }
}
```


AnimeViewModel
```
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch

class AnimeViewModel : ViewModel() {
    private val repository = AnimeRepository()

    val images = mutableStateListOf<String>()
    val isLoading = mutableStateOf(false)
    val error = mutableStateOf<String?>(null)

    fun openAnimeImage(category: String, count: Int) {
        viewModelScope.launch {
            try {
                isLoading.value = true
                val newImages = repository.getAnime(category, count)
                images.clear()
                images.addAll(newImages)
                error.value = null
            } catch (e: Exception) {
                error.value = "Ошибка: ${e.message}"
            } finally {
                isLoading.value = false
            }
        }
    }
}
```


AnimeScreen
```
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.Button
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.RadioButton
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import coil3.compose.AsyncImage
import coil3.compose.rememberAsyncImagePainter

@Composable
fun AnimeScreen(viewModel: AnimeViewModel = viewModel()) {
    val categories = listOf("neko", "waifu", "husbando", "kitsune")
    var selectedCategory by remember { mutableStateOf(categories[0]) }
    var count by remember { mutableStateOf("5") }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("Выберите категорию:", style = MaterialTheme.typography.titleMedium)
        categories.forEach { category ->
            Row(verticalAlignment = Alignment.CenterVertically) {
                RadioButton(
                    selected = selectedCategory == category,
                    onClick = { selectedCategory = category }
                )
                Text(category)
            }
        }

        OutlinedTextField(
            value = count,
            onValueChange = { count = it },
            label = { Text("Количество изображений") },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number)
        )

        Button(
            onClick = { viewModel.openAnimeImage(selectedCategory, count.toIntOrNull() ?: 1) },
            modifier = Modifier.padding(vertical = 8.dp)
        ) {
            Text("Загрузить")
        }

        if (viewModel.isLoading.value) {
            CircularProgressIndicator()
        }

        viewModel.error.value?.let { error ->
            Text(error, color = MaterialTheme.colorScheme.error)
        }

        LazyColumn {
            items(viewModel.images) { url ->
                AsyncImage(
                    model = url,
                    contentDescription = null,
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(8.dp),
                    contentScale = ContentScale.Crop
                )
            }
        }
    }
}
```


MainActivity
```
import android.media.Image
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.viewModels
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.animegirls.ui.theme.AnimeGirlsTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            AnimeGirlsTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    val viewModel: AnimeViewModel = viewModel()
                    AnimeScreen(viewModel)
                }
            }
        }
    }
}
```
