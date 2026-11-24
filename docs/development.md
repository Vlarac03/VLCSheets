# Development

**English** · [Català](development.ca.md) · [Español](development.es.md)

Back to the [README](../README.md).

## Architecture

```
app/src/main/java/com/vlcsheets/app
├── MainActivity.kt          Edge-to-edge, pedal keys, VIEW intents
├── MainViewModel.kt         Library state, filters, actions, metronome
├── VlcSheetsApp.kt          Manual dependency container
├── data
│   ├── Models.kt            Song, Setlist, Collection, Stroke, Bookmark + JSON
│   ├── LibraryRepository.kt Atomic JSON index, import, backup, restore, audio
│   ├── ChordPro.kt          Text and ChordPro parsing
│   ├── Handwriting.kt       Handwriting recogniser (point cloud, $P)
│   ├── MsfReader.kt         Reads .msf files
│   └── SettingsRepository.kt Reactive SharedPreferences
├── media
│   ├── Media.kt             PdfRenderer, BitmapFactory, EXIF, margin cropping, thumbnails
│   ├── AudioPlayer.kt       Plays each score's audio (MediaPlayer)
│   ├── PdfExporter.kt       PDF with annotations, written page by page
│   ├── ChordProPdf.kt       Text and ChordPro turned into PDF
│   ├── Practice.kt          Audio marks, loop and metronome page turns
│   ├── Tuner.kt             Pitch detection (YIN)
│   └── MetronomeEngine.kt   Click generation on AudioTrack
└── ui
    ├── App.kt               Navigation, snackbars, bottom bar
    ├── theme/Theme.kt       Palette, typography, shapes (Material 3)
    ├── components/Common.kt Brand mark, thumbnails, empty states, dialogs
    └── screens
        ├── LibraryScreen.kt      Grid/list, filters, import
        ├── SongEditorSheet.kt    Metadata and collections
        ├── SetlistsScreen.kt     Setlists, detail and score picker
        ├── CollectionsScreen.kt  Collections
        ├── SettingsScreen.kt     Settings, backup, about
        ├── HandwritingTrainer.kt Teaching your handwriting
        ├── TunerSheet.kt         Tuner
        └── ViewerScreen.kt       Viewer, zoom, annotations, bookmarks, metronome, audio
```

**No external database or ORM.** The library is a small, atomic `index.json`
(write to a temp file + `fsync` + rename) inside `filesDir/library`, next to the
PDFs. Each score's annotations live in their own file (`songs/<id>/annotations.json`),
so a new stroke does not rewrite the whole index. A backup is just a zipped folder,
and the app starts instantly.

**No third-party PDF libraries.** Rendering uses `android.graphics.pdf.PdfRenderer`.
Pages are rendered at about 1.4× the screen width in `ARGB_8888`, the only format
`PdfRenderer` accepts; images are decoded in `RGB_565` to save memory.

## Build and test

Requirements: JDK 17+ and the Android SDK with platform 36.

```bash
# 1. Point to the SDK
echo "sdk.dir=C\:\\Android\\sdk" > local.properties

# 2. Build
./gradlew :app:assembleDebug

# 3. Tests and lint
./gradlew :app:testDebugUnitTest
./gradlew :app:lintDebug

# 4. Install
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

The APK is `app/build/outputs/apk/debug/app-debug.apk`.
