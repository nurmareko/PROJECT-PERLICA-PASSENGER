# Airport Taxi Sharing — passenger app

Flutter passenger client for the Airport Taxi Sharing service. Android is the
validated build target on Windows; the presence of other platform folders does
not mean the mobile integrations work on those platforms.

## Development toolchain

Upgraded on 2026-10-26 using Flutter's stable channel and its Android template
versions as the compatibility baseline:

| Component | Version |
| --- | --- |
| Flutter | 3.47.8 stable |
| Dart (bundled with Flutter) | 3.13.7 |
| JDK | 17 (validated with Temurin 17.0.20.101) |
| Android Gradle Plugin | 9.1.0 |
| Gradle wrapper | 9.3.1 |
| Kotlin Gradle Plugin | 2.4.0 |
| Google Services Gradle plugin | 4.5.0 |
| Android compile SDK | 37 (SDK Manager package `platforms;android-37.0`) |
| Android target SDK | 36 (Flutter default) |
| Android minimum SDK | 24 / Android 7.0 (Flutter default) |
| Android NDK | 28.2.13676358 (Flutter default) |

Use the project's `pubspec.lock` to reproduce Dart dependencies. Gradle is
provided by the project wrapper; a separate global Gradle installation is not
needed. Java 17 is the compatible build baseline, not the newest Java release.

### Windows setup

1. Install Git, Flutter stable, a JDK 17 distribution, and Android Studio with
   the Android SDK command-line tools, platform tools, emulator, and a Google
   Play system image (or connect a physical Android device with USB debugging).
2. Enable **Developer Mode** in Windows Settings. Flutter requires permission
   to create plugin symlinks, even when only building Android from this
   multi-platform project.
3. Point Flutter to your JDK 17 installation with `flutter config --jdk-dir=...`.
   This setting affects all projects using that Flutter installation. Set
   `ANDROID_HOME` to your Android SDK installation if it is not detected.
4. Review SDK licenses and verify the toolchain:

   ```sh
   flutter doctor --android-licenses
   flutter doctor -v
   flutter pub get
   ```

Chrome and Visual Studio are not required for the Android target. Their doctor
warnings only matter when developing web or Windows desktop targets. iOS builds
require macOS and Xcode.

## Build and run

```sh
flutter build apk --debug
flutter emulators
flutter emulators --launch Medium_Phone_API_36.1
flutter devices
flutter run -d <android-device-id>
```

The emulator name above is the existing emulator on the setup machine; use the
name returned by `flutter emulators` on other machines. Replace
`<android-device-id>` with the ID returned by `flutter devices`.

Debug APK: `build/app/outputs/flutter-apk/app-debug.apk`.

The existing Firebase configuration, Google Maps configuration, and remote API
endpoint are retained. A successful build does not verify backend availability,
Maps billing/key restrictions, push notifications, or authenticated flows. Do
not use the current debug signing configuration for a production release.

## Upgrade decisions

- Direct dependencies were updated to the latest stable versions resolved by
  Pub, with one intentional exception: **`flutter_secure_storage` stays on
  `^10.3.4`**. The original app used version 9's legacy encryption algorithms.
  Version 11 removes those algorithms and cannot directly migrate existing
  credentials. Version 10 retains automatic migration support. Test an in-place
  upgrade with existing credentials before shipping; do not move to v11 until
  users who skip intermediate releases are also handled safely.
- `android.builtInKotlin=false` and `android.newDsl=false` remain enabled as
  compatibility opt-outs. The current `app_settings`, `firebase_core`, and
  `location` plugins still apply the Kotlin Gradle Plugin. Do not remove these
  flags until the whole Android plugin graph supports the migration.
- Compile SDK 37 is required by `permission_handler_android` 14.1.0. Target SDK
  remains the Flutter default, 36; changing the compile SDK alone does not opt
  into new target-SDK runtime behavior.
- Notification, reverse-geocoding, polyline, and Google Fonts call sites were
  adapted to the upgraded APIs without changing the app architecture.
- Some transitive packages remain below their newest published versions because
  Flutter or their parent packages constrain them. Do not force dependency
  overrides merely to make `flutter pub outdated` empty.

## Validation status (2026-10-26)

- `flutter build apk --debug --no-pub`: **passed**.
- `flutter analyze --no-pub`: **0 errors**, 9 warnings and 186 informational
  diagnostics. The analyzer still exits nonzero; existing diagnostics were not
  suppressed or removed for this upgrade.
- `flutter test --no-pub`: **fails** in the existing `test/widget_test.dart`,
  which expects the starter counter screen rather than this app's splash screen.
  The test was left intact, not skipped.
- Runtime emulator, backend, notification, and credential-migration checks have
  **not** been performed.
- Plugin changes still triggered a Windows symlink-permission error during
  `flutter pub get` on the setup machine. Enable Developer Mode and rerun it;
  the successful Android build used already-resolved dependencies.

Before editing, a source backup was saved beside the project as
`passenger-pre-upgrade-20261026-015135.zip` because the workspace had no Git
repository. It contains the original project configuration; keep it private.

## References

- [Flutter stable SDK upgrades](https://docs.flutter.dev/install/upgrade)
- [Flutter 3.47.8 Android version defaults](https://github.com/flutter/flutter/blob/737ba8f885dc774b53b442759999a312c9461e4e/packages/flutter_tools/lib/src/android/gradle_utils.dart)
- [Flutter built-in Kotlin migration](https://docs.flutter.dev/release/breaking-changes/migrate-to-built-in-kotlin/for-app-developers)
- [Local notifications Android setup](https://pub.dev/packages/flutter_local_notifications#android-setup)
- [Secure storage changelog and migration notes](https://pub.dev/packages/flutter_secure_storage/changelog)
