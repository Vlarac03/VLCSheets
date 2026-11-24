# Desenvolupament

[English](development.md) · **Català** · [Español](development.es.md)

Torna al [README](../README.ca.md).

## Arquitectura

```
app/src/main/java/com/vlcsheets/app
├── MainActivity.kt          Edge-to-edge, gestió de tecles de pedal, intents VIEW
├── MainViewModel.kt         Estat de la biblioteca, filtres, accions, metrònom
├── VlcSheetsApp.kt          Contenidor d'injecció manual
├── data
│   ├── Models.kt            Song, Setlist, Collection, Stroke, Bookmark + JSON
│   ├── LibraryRepository.kt Índex JSON atòmic, importació, còpia, restauració, àudios
│   ├── ChordPro.kt          Lectura de text i ChordPro
│   ├── Handwriting.kt       Reconeixedor de lletra a mà (núvol de punts, $P)
│   ├── MsfReader.kt         Lectura dels .msf
│   └── SettingsRepository.kt SharedPreferences reactiu
├── media
│   ├── Media.kt             PdfRenderer, BitmapFactory, EXIF, retall de marges, miniatures
│   ├── AudioPlayer.kt       Reproductor dels àudios de cada partitura (MediaPlayer)
│   ├── PdfExporter.kt       PDF amb les anotacions, pàgina a pàgina
│   ├── ChordProPdf.kt       Text i ChordPro convertits en PDF
│   ├── Practice.kt          Punts d'àudio, bucle i pas de pàgina del metrònom
│   ├── Tuner.kt             Detecció de la nota (mètode YIN)
│   └── MetronomeEngine.kt   Generació de clics amb AudioTrack
└── ui
    ├── App.kt               Navegació, snackbars, barra inferior
    ├── theme/Theme.kt       Paleta, tipografia, formes (Material 3)
    ├── components/Common.kt Marca, miniatures, estats buits, diàlegs
    └── screens
        ├── LibraryScreen.kt      Graella/llista, filtres, importació
        ├── SongEditorSheet.kt    Edició de metadades i col·leccions
        ├── SetlistsScreen.kt     Llistes + detall + selector de cançons
        ├── CollectionsScreen.kt  Col·leccions
        ├── SettingsScreen.kt     Ajustos, còpia de seguretat, quant a
        ├── HandwritingTrainer.kt Ensenyar la lletra a mà
        ├── TunerSheet.kt         Afinador
        └── ViewerScreen.kt       Visor, zoom, anotacions, marcadors, metrònom, àudio
```

**Sense base de dades externa ni ORM.** La biblioteca és un `index.json` petit i
atòmic (escriptura a temporal + `fsync` + rename) dins `filesDir/library`, al costat
dels PDFs. Les anotacions de cada partitura van a part (`songs/<id>/annotations.json`),
perquè un traç nou no hagi de reescriure tot l'índex. Això fa que la còpia de seguretat sigui només comprimir una carpeta i que
l'app arrenqui a l'instant.

**Sense llibreries de PDF de tercers.** El rasteritzat usa `android.graphics.pdf.PdfRenderer`.
Les pàgines es renderitzen a unes 1,4 vegades l'amplada de la pantalla
en `ARGB_8888`, l'únic format que accepta `PdfRenderer`; les imatges es descodifiquen
en `RGB_565` per gastar menys memòria.

## Compilar i provar

Requisits: JDK 17+ i Android SDK amb la plataforma 36.

```bash
# 1. Configura el SDK
echo "sdk.dir=C\:\\Android\\sdk" > local.properties

# 2. Compila
./gradlew :app:assembleDebug

# 3. Proves i lint
./gradlew :app:testDebugUnitTest
./gradlew :app:lintDebug

# 4. Instal·la
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

L'APK resultant és `app/build/outputs/apk/debug/app-debug.apk`.
