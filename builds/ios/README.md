## Helm for iOS (AUv3)

`builds/ios/Helm_iOS.xcodeproj` builds Helm as an **AUv3 audio unit app
extension** (`Helm.appex`) together with the container app (`Helm.app`) that
hosts it, both targeting **iPadOS 17.0**.

The project is generated — edit `generate_project.py`, not the `.pbxproj` —
because this tree ships JUCE 5.3.2 without the Projucer sources that normally
create and maintain Xcode projects:

```bash
python3 builds/ios/generate_project.py
```

Re-run it whenever files are added to `src/`, `mopo/src/` or the
`JuceLibraryCode` include list.

### Targets

| Target | Product | Purpose |
| --- | --- | --- |
| `Helm - Shared Code` | `libHelm.a` | All of Helm, `mopo` and the JUCE modules |
| `Helm - AUv3` | `Helm.appex` | AUv3 app extension (`JucePlugin_Build_AUv3=1`) |
| `Helm - AUv3 App` | `Helm.app` | Standalone host that embeds the appex (`JucePlugin_Build_Standalone=1`) |

### Building locally

```bash
xcodebuild -project builds/ios/Helm_iOS.xcodeproj \
  -target "Helm - AUv3 App" -configuration Release -sdk iphoneos build
```

To run on a device, open the project in Xcode, select your team under
*Signing & Capabilities* for both targets (a free Apple ID works), then run.

### CI

`.github/workflows/ios-auv3.yml` builds on `macos-15`, verifies that the
generated project is committed in sync, checks that the appex contains the
`helmFactoryAUv3` factory, the `com.apple.AudioUnit-UI` extension point and
the factory patches, and uploads:

- `Helm-AUv3-iOS-unsigned.ipa` – unsigned `Payload/Helm.app` archive
- `Helm-AUv3-iOS.appex.zip` – the bare app extension
- `build.log`

Unsigned builds cannot be installed directly. Sign the IPA with your own
certificate (Sideloadly, AltStore, iOS App Signer, or `codesign` inside Xcode)
before installing. Once installed, enable **Helm** as an Audio Unit in an
AUv3 host such as AUM, Cubasis or GarageBand.

### iOS-specific behaviour

- **Patches** – the factory bank is read from `Helm.appex/patches`. On first
  launch it is copied into `Documents/Helm/Patches`, which is where user
  patches are read and written. `UIFileSharingEnabled` is on, so the standalone
  app's patch folder is visible in the Files app.
- **Audio unit** – `aumu` / manufacturer `Tyte` / subtype `Helm`, so existing
  Helm presets and MIDI maps keep working.
- **Graphics** – the bundled shaders are GLSL ES 1.00, so the editor requests
  an ES2 OpenGL context on iOS.
- **Editor size** – the desktop window size config is ignored; the editor is
  sized to fit the display and then scaled by the host.
- **Not available on iOS** – desktop update check, the donation nag, and the
  `~/.helm` style legacy patch migration.

### Known limitations

- Only arm64 device builds are produced; a simulator build is not part of CI.
- Audio Unit validation (`auval`) requires a signed install and is not run.
- The plug-in is GPLv3: distributing a binary requires providing the source.
