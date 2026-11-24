# Desarrollo

[English](development.md) · [Català](development.ca.md) · **Español**

Vuelve al [README](../README.es.md).

## Arquitectura

```
app/src/main/java/com/vlcsheets/app
├── MainActivity.kt          Edge-to-edge, gestión de teclas de pedal, intents VIEW
├── MainViewModel.kt         Estado de la biblioteca, filtros, acciones, metrónomo
├── VlcSheetsApp.kt          Contenedor de inyección manual
├── data
│   ├── Models.kt            Song, Setlist, Collection, Stroke, Bookmark + JSON
│   ├── LibraryRepository.kt Índice JSON atómico, importación, copia, restauración, audios
│   ├── ChordPro.kt          Lectura de texto y ChordPro
│   ├── Handwriting.kt       Reconocedor de letra a mano (nube de puntos, $P)
│   ├── MsfReader.kt         Lectura de los .msf
│   └── SettingsRepository.kt SharedPreferences reactivo
├── media
│   ├── Media.kt             PdfRenderer, BitmapFactory, EXIF, recorte de márgenes, miniaturas
│   ├── AudioPlayer.kt       Reproductor de los audios de cada partitura (MediaPlayer)
│   ├── PdfExporter.kt       PDF con las anotaciones, página a página
│   ├── ChordProPdf.kt       Texto y ChordPro convertidos en PDF
│   ├── Practice.kt          Puntos de audio, bucle y paso de página del metrónomo
│   ├── Tuner.kt             Detección de la nota (método YIN)
│   └── MetronomeEngine.kt   Generación de clics con AudioTrack
└── ui
    ├── App.kt               Navegación, snackbars, barra inferior
    ├── theme/Theme.kt       Paleta, tipografía, formas (Material 3)
    ├── components/Common.kt Marca, miniaturas, estados vacíos, diálogos
    └── screens
        ├── LibraryScreen.kt      Cuadrícula/lista, filtros, importación
        ├── SongEditorSheet.kt    Edición de metadatos y colecciones
        ├── SetlistsScreen.kt     Listas + detalle + selector de canciones
        ├── CollectionsScreen.kt  Colecciones
        ├── SettingsScreen.kt     Ajustes, copia de seguridad, acerca de
        ├── HandwritingTrainer.kt Enseñar la letra a mano
        ├── TunerSheet.kt         Afinador
        └── ViewerScreen.kt       Visor, zoom, anotaciones, marcadores, metrónomo, audio
```

**Sin base de datos externa ni ORM.** La biblioteca es un `index.json` pequeño y
atómico (escritura a un temporal + `fsync` + renombrar) dentro de `filesDir/library`,
junto a los PDF. Las anotaciones de cada partitura van aparte
(`songs/<id>/annotations.json`), para que un trazo nuevo no reescriba todo el índice.
La copia de seguridad es solo comprimir una carpeta y la app arranca al instante.

**Sin librerías de PDF de terceros.** El renderizado usa `android.graphics.pdf.PdfRenderer`.
Las páginas se renderizan a unas 1,4 veces el ancho de la pantalla en `ARGB_8888`, el
único formato que acepta `PdfRenderer`; las imágenes se decodifican en `RGB_565` para
gastar menos memoria.

## Compilar y probar

Requisitos: JDK 17+ y Android SDK con la plataforma 36.

```bash
# 1. Configura el SDK
echo "sdk.dir=C\:\\Android\\sdk" > local.properties

# 2. Compila
./gradlew :app:assembleDebug

# 3. Pruebas y lint
./gradlew :app:testDebugUnitTest
./gradlew :app:lintDebug

# 4. Instala
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

El APK queda en `app/build/outputs/apk/debug/app-debug.apk`.
