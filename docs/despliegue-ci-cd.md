# Despliegue y CI/CD en TecnoTime

## Índice
1. [Introducción](#introducción)
2. [Configuración de Build con Gradle](#configuración-de-build-con-gradle)
3. [Signing de APK](#signing-de-apk)
4. [CI/CD con GitHub Actions](#cicd-con-github-actions)
5. [Estrategia de Versionado](#estrategia-de-versionado)
6. [Release Management](#release-management)
7. [Distribución](#distribución)
8. [Monitoreo y Analytics](#monitoreo-y-analytics)
9. [Rollback Strategy](#rollback-strategy)
10. [Referencias Cruzadas](#referencias-cruzadas)

## Introducción

TecnoTime implementa un proceso de despliegue automatizado que asegura calidad, consistencia y rapidez en la entrega de nuevas versiones. Utilizando Gradle para build management, GitHub Actions para CI/CD, y estrategias de signing seguro, el proceso permite releases frecuentes con mínima intervención manual.

Este documento detalla la configuración técnica y mejores prácticas para el despliegue de TecnoTime.

## Configuración de Build con Gradle

### Estructura del Proyecto Gradle

```
app/
├── build.gradle.kts          # Configuración principal
├── proguard-rules.pro        # Reglas de ofuscación
├── src/main/AndroidManifest.xml
├── src/main/res/
└── src/main/java/
```

### Build Gradle Principal

```kotlin
// app/build.gradle.kts
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("kotlin-kapt")
    id("dagger.hilt.android.plugin")
    id("kotlin-parcelize")
    id("androidx.navigation.safeargs.kotlin")
}

android {
    namespace = "com.example.tecnotime"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.tecnotime"
        minSdk = 24
        targetSdk = 35
        versionCode = 1
        versionName = "3.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            buildConfigField("String", "API_BASE_URL", "\"https://api.tecnotime.com\"")
            buildConfigField("Boolean", "ENABLE_CRASH_REPORTING", "true")
        }

        debug {
            buildConfigField("String", "API_BASE_URL", "\"https://dev-api.tecnotime.com\"")
            buildConfigField("Boolean", "ENABLE_CRASH_REPORTING", "false")
            isDebuggable = true
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.4.3"
    }

    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    // Core Android
    implementation("androidx.core:core-ktx:1.10.1")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.6.1")

    // Compose
    implementation("androidx.compose.ui:ui:1.4.3")
    implementation("androidx.compose.material3:material3:1.1.0")
    implementation("androidx.compose.ui:ui-tooling-preview:1.4.3")

    // Navigation
    implementation("androidx.navigation:navigation-compose:2.6.0")

    // Room
    implementation("androidx.room:room-runtime:2.5.2")
    implementation("androidx.room:room-ktx:2.5.2")
    kapt("androidx.room:room-compiler:2.5.2")

    // Hilt
    implementation("com.google.dagger:hilt-android:2.45")
    kapt("com.google.dagger:hilt-compiler:2.45")

    // Firebase
    implementation(platform("com.google.firebase:firebase-bom:32.0.0"))
    implementation("com.google.firebase:firebase-auth-ktx")
    implementation("com.google.firebase:firebase-firestore-ktx")

    // Jsoup para scraping
    implementation("org.jsoup:jsoup:1.16.1")

    // PDF processing
    implementation("com.github.barteksc:android-pdf-viewer:3.2.0-beta.1")
    implementation("com.tom_roush:pdfbox-android:2.0.27.0")

    // WorkManager
    implementation("androidx.work:work-runtime-ktx:2.8.1")

    // Testing
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.1")
    testImplementation("io.mockk:mockk:1.13.4")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4:1.4.3")
}
```

### Configuración de Product Flavors

```kotlin
android {
    flavorDimensions += "environment"

    productFlavors {
        create("development") {
            dimension = "environment"
            applicationIdSuffix = ".dev"
            versionNameSuffix = "-dev"
            buildConfigField("String", "BASE_URL", "\"https://dev-api.tecnotime.com\"")
        }

        create("staging") {
            dimension = "environment"
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-staging"
            buildConfigField("String", "BASE_URL", "\"https://staging-api.tecnotime.com\"")
        }

        create("production") {
            dimension = "environment"
            buildConfigField("String", "BASE_URL", "\"https://api.tecnotime.com\"")
        }
    }
}
```

### Reglas de ProGuard

```proguard
# proguard-rules.pro

# Keep data classes
-keep class com.example.tecnotime.domain.model.** { *; }

# Keep Hilt generated classes
-keep class dagger.hilt.** { *; }
-keep class javax.inject.** { *; }
-keep class * extends dagger.hilt.android.HiltAndroidApp

# Keep Room entities
-keep @androidx.room.Entity class *
-keep class * extends androidx.room.RoomDatabase

# Keep Compose classes
-keep class androidx.compose.** { *; }

# Keep Firebase classes
-keep class com.google.firebase.** { *; }

# Obfuscation rules for Jsoup
-keep class org.jsoup.** { *; }

# PDFBox rules
-keep class org.apache.pdfbox.** { *; }
-dontwarn org.apache.pdfbox.**

# General rules
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations
-keepattributes AnnotationDefault

-renamesourcefileattribute SourceFile
-keepattributes SourceFile,LineNumberTable
```

## Signing de APK

### Configuración de Signing

```kotlin
// app/build.gradle.kts
android {
    signingConfigs {
        create("release") {
            storeFile = file("path/to/keystore.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }

        create("ci") {
            storeFile = file("ci-keystore.jks")
            storePassword = "ci_password"
            keyAlias = "ci_key"
            keyPassword = "ci_password"
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            // ... otras configuraciones
        }

        debug {
            signingConfig = signingConfigs.getByName("ci")
        }
    }
}
```

### Gestión Segura de Credenciales

#### Opción 1: Variables de Entorno (CI)

```bash
# En GitHub Actions
KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
```

#### Opción 2: Keystore en Repositorio (CI Only)

```kotlin
// Solo para builds de CI
android {
    signingConfigs {
        create("ci") {
            storeFile = rootProject.file("ci-keystore.jks")
            storePassword = "ci_store_password"
            keyAlias = "ci_key_alias"
            keyPassword = "ci_key_password"
        }
    }
}
```

### Generación de Keystore

```bash
# Generar keystore para release
keytool -genkey -v -keystore release.keystore \
  -alias tecnotime-key \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -dname "CN=TecnoTime, OU=UMSS, O=UMSS, L=Cochabamba, ST=Cochabamba, C=BO"
```

## CI/CD con GitHub Actions

### Workflow Principal de CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-

    - name: Grant execute permission for gradlew
      run: chmod +x gradlew

    - name: Run unit tests
      run: ./gradlew testDebugUnitTest --no-daemon

    - name: Run instrumentation tests
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: 29
        target: google_apis
        arch: x86_64
        profile: Nexus 6
        script: ./gradlew connectedDebugAndroidTest --no-daemon

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results
        path: |
          app/build/reports/tests/
          app/build/outputs/androidTest-results/

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: app/build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml
        flags: unittests
        name: codecov-umbrella

  build:
    needs: test
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

    - name: Build debug APK
      run: ./gradlew assembleDebug --no-daemon

    - name: Build release APK
      run: ./gradlew assembleRelease --no-daemon
      env:
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}

    - name: Upload APKs
      uses: actions/upload-artifact@v3
      with:
        name: apks
        path: |
          app/build/outputs/apk/debug/*.apk
          app/build/outputs/apk/release/*.apk
```

### Workflow de Release

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: ~/.gradle/caches
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

    - name: Build release APK
      run: ./gradlew assembleRelease --no-daemon
      env:
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}

    - name: Create release
      uses: softprops/action-gh-release@v1
      with:
        files: app/build/outputs/apk/release/*.apk
        generate_release_notes: true
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Workflow de Deploy a Play Store

```yaml
# .github/workflows/play-store.yml
name: Deploy to Play Store

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 11
      uses: actions/setup-java@v4
      with:
        java-version: '11'
        distribution: 'temurin'

    - name: Build release bundle
      run: ./gradlew bundleRelease --no-daemon
      env:
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}

    - name: Deploy to Play Store (Beta)
      uses: r0adkll/upload-google-play@v1
      with:
        serviceAccountJsonPlainText: ${{ secrets.SERVICE_ACCOUNT_JSON }}
        packageName: com.example.tecnotime
        releaseFiles: app/build/outputs/bundle/release/*.aab
        track: beta
        inAppUpdatePriority: 3
        whatsNewDirectory: distribution/whatsnew
        mappingFile: app/build/outputs/mapping/release/mapping.txt
```

## Estrategia de Versionado

### Version Code y Version Name

```kotlin
// app/build.gradle.kts
defaultConfig {
    versionCode = 1  // Incrementa en cada release
    versionName = "3.0"  // Versión semántica
}
```

### Versionado Semántico

- **Major**: Cambios incompatibles (3.0.0)
- **Minor**: Nuevas funcionalidades (3.1.0)
- **Patch**: Bug fixes (3.0.1)

### Automatización de Versionado

```kotlin
// version.properties
version.major=3
version.minor=0
version.patch=0
version.build=1

// build.gradle.kts
val versionProps = Properties().apply {
    load(rootProject.file("version.properties").inputStream())
}

android {
    defaultConfig {
        versionCode = versionProps.getProperty("version.build").toInt()
        versionName = "${versionProps.getProperty("version.major")}.${versionProps.getProperty("version.minor")}.${versionProps.getProperty("version.patch")}"
    }
}
```

## Release Management

### Checklist de Release

- [ ] Todos los tests pasan
- [ ] Cobertura de código > 75%
- [ ] No hay warnings de lint críticos
- [ ] APK firmado correctamente
- [ ] Release notes actualizados
- [ ] Screenshots actualizados en Play Store
- [ ] Privacy policy actualizada si aplica

### Release Notes Template

```markdown
## Version 3.0.1

### Bug Fixes
- Corregido crash en scraping de horarios
- Mejorado manejo de errores de red

### Improvements
- Optimización de rendimiento en generación de horarios
- Mejor UX en pantalla de configuración

### Technical
- Actualización de dependencias
- Mejoras en cobertura de tests
```

### Branching Strategy

```
main (production)
├── develop (integration)
│   ├── feature/scraping-improvements
│   ├── feature/ui-redesign
│   └── hotfix/crash-fix
└── release/v3.0.1 (release candidate)
```

## Distribución

### Google Play Store

#### Configuración de App Bundle

```kotlin
// app/build.gradle.kts
android {
    bundle {
        language {
            enableSplit = true
        }
        density {
            enableSplit = true
        }
        abi {
            enableSplit = true
        }
    }
}
```

#### Fastlane para Automation

```ruby
# fastlane/Fastfile
lane :deploy_beta do
  upload_to_play_store(
    track: 'beta',
    aab: 'app/build/outputs/bundle/release/app-release.aab',
    skip_upload_metadata: true,
    skip_upload_changelogs: true,
    skip_upload_images: true,
    skip_upload_screenshots: true
  )
end
```

### Distribución Interna

#### Firebase App Distribution

```yaml
# .github/workflows/firebase-distribution.yml
name: Firebase Distribution

on:
  push:
    branches: [ develop ]

jobs:
  distribute:
    runs-on: ubuntu-latest

    steps:
    - name: Build APK
      run: ./gradlew assembleDebug

    - name: Upload to Firebase Distribution
      uses: wzieba/Firebase-Distribution-Github-Action@v1
      with:
        appId: ${{ secrets.FIREBASE_APP_ID }}
        token: ${{ secrets.FIREBASE_TOKEN }}
        groups: testers
        file: app/build/outputs/apk/debug/app-debug.apk
```

## Monitoreo y Analytics

### Firebase Crashlytics

```kotlin
// Application class
class TecnoTimeApp : Application() {

    override fun onCreate() {
        super.onCreate()

        if (BuildConfig.ENABLE_CRASH_REPORTING) {
            FirebaseApp.initializeApp(this)
            FirebaseCrashlytics.getInstance().setCrashlyticsCollectionEnabled(true)
        }
    }
}
```

### Google Analytics

```kotlin
// build.gradle.kts
implementation("com.google.firebase:firebase-analytics-ktx")

// Usage
FirebaseAnalytics.getInstance(this).logEvent("schedule_generated") {
    param("subject_count", subjectCount.toLong())
    param("generation_time", durationMs)
}
```

### Custom Metrics

- Tasa de éxito de scraping
- Tiempo promedio de generación de horarios
- Número de crashes por versión
- Uso de funcionalidades por usuario

## Rollback Strategy

### Play Store Rollback

1. **Pausar rollout** en Play Store Console
2. **Revertir código** a versión anterior
3. **Crear hotfix release** con corrección
4. **Gradual rollout** de la corrección

### Database Migration Rollback

```kotlin
// Migration strategy
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Forward migration
        database.execSQL("ALTER TABLE subjects ADD COLUMN is_elective INTEGER NOT NULL DEFAULT 0")
    }
}

val MIGRATION_2_1 = object : Migration(2, 1) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Rollback migration (if safe)
        database.execSQL("ALTER TABLE subjects DROP COLUMN is_elective")
    }
}
```

### Feature Flags

```kotlin
// Remote config for feature toggles
val isNewScrapingEnabled = FirebaseRemoteConfig.getInstance()
    .getBoolean("new_scraping_enabled")

if (isNewScrapingEnabled) {
    // Use new scraping logic
} else {
    // Use old scraping logic
}
```

## Referencias Cruzadas

- [docs/arquitectura.md](docs/arquitectura.md): Arquitectura del sistema
- [docs/testing-estrategias.md](docs/testing-estrategias.md): Testing en CI/CD
- [docs/decisiones-diseno-adr.md](docs/decisiones-diseno-adr.md): Decisiones de despliegue
- [docs/funcionalidades.md](docs/funcionalidades.md): Funcionalidades desplegadas

El proceso de despliegue de TecnoTime combina automatización, calidad y velocidad, permitiendo releases frecuentes con confianza y trazabilidad completa.