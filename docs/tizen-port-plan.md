# Samsung Tizen Port Plan

## Objective

Add Samsung Tizen TV support to PlxNative without disrupting the existing LG webOS application.

The Tizen target should be implemented as a separate web application packaged as a signed `.wgt`. The existing LG target remains an LG webOS `.ipk` application using its current Rust, SDL/GLES, Starfish, and webOS-specific code.

## Architecture decision

Do not attempt to run the existing LG binary on Samsung. Samsung TVs use Tizen and different application, media, input, packaging, and signing APIs.

Create a separate Tizen frontend and progressively share platform-independent Plex/domain logic where practical.

Initial mapping:

| LG webOS target | Samsung Tizen target |
|---|---|
| Rust/native application | TypeScript/browser application |
| StarfishMediaAPIs | Samsung AVPlay API |
| SDL/GLES renderer | DOM/CSS first; WebGL only if needed |
| LG remote/FIFO | Tizen keyboard and TV input events |
| `.ipk` package | signed `.wgt` package |
| webOS Developer Mode/Homebrew | Tizen TV Developer Mode/SDB |

## Proposed repository layout

```text
tizen/
├── app/
│   ├── config.xml
│   ├── index.html
│   ├── icon.png
│   ├── css/
│   │   ├── app.css
│   │   ├── focus.css
│   │   └── player.css
│   ├── src/
│   │   ├── main.ts
│   │   ├── app.ts
│   │   ├── router.ts
│   │   ├── focus/
│   │   │   ├── focus-engine.ts
│   │   │   └── focus-rect.ts
│   │   ├── plex/
│   │   │   ├── plex-client.ts
│   │   │   ├── plex-auth.ts
│   │   │   ├── plex-models.ts
│   │   │   └── plex-playback.ts
│   │   ├── playback/
│   │   │   ├── player.ts
│   │   │   ├── avplay-player.ts
│   │   │   ├── html5-player.ts
│   │   │   ├── subtitles.ts
│   │   │   └── playback-state.ts
│   │   ├── screens/
│   │   │   ├── login-screen.ts
│   │   │   ├── home-screen.ts
│   │   │   ├── library-screen.ts
│   │   │   ├── detail-screen.ts
│   │   │   ├── search-screen.ts
│   │   │   └── player-screen.ts
│   │   ├── input/
│   │   │   └── samsung-remote.ts
│   │   └── platform/
│   │       ├── tizen-platform.ts
│   │       ├── storage.ts
│   │       └── diagnostics.ts
│   └── webapis/
│       └── webapis.d.ts
├── scripts/
│   ├── tizen-build.sh
│   ├── tizen-package.sh
│   ├── tizen-install.sh
│   ├── tizen-run.sh
│   ├── tizen-uninstall.sh
│   ├── tizen-log.sh
│   └── tizen-device-check.sh
├── tests/
│   └── tizen/
│       ├── plex-client.test.ts
│       ├── focus-engine.test.ts
│       └── playback-plan.test.ts
├── package.json
└── Makefile.tizen
```

Keep LG code in its existing locations. Do not add Samsung conditionals to Starfish, ACB, LG launcher, or webOS-specific modules.

## Implementation phases

### Phase 0: repository and build shell

Deliverables:

- `tizen/` application directory
- Tizen `config.xml`
- TypeScript compiler/bundler setup
- Minimal 1920x1080 placeholder screen
- `package.json` scripts
- Tizen build, package, install, run, uninstall, log, and device-check scripts
- Development documentation
- CI validation for TypeScript and package layout
- No changes to the existing LG build

Acceptance criteria:

```sh
npm run tizen:build
npm run tizen:package
```

produces a signed `.wgt` when a local certificate profile is configured.

### Phase 1: Tizen runtime shell

Deliverables:

- Full-screen app bootstrap
- Screen router
- Focus manager
- Error overlay
- Diagnostic logging
- Persistent settings
- Samsung remote adapter
- Back-button handling
- App lifecycle handling

Acceptance criteria:

- App launches on a physical Samsung TV.
- Arrow, Enter, and Back work.
- App can be closed and relaunched.
- Logs can be collected over SDB.

### Phase 2: Plex authentication and browsing

Deliverables:

- Plex PIN authentication
- QR-code or short-link login screen
- Token persistence and logout/reset
- Plex resource/server discovery
- Plex Home profile selection
- Home screen
- Library grid
- Search
- Metadata and detail screen

Acceptance criteria:

- A user can authenticate without manually entering a server address.
- The app can browse libraries and search metadata from a real Plex account.
- Tokens are never committed or printed in CI logs.

### Phase 3: AVPlay playback

Deliverables:

- Platform-neutral `Player` interface
- Samsung `AvplayPlayer` implementation
- Browser `Html5Player` fallback for development
- Open/prepare/play/pause/stop lifecycle
- Position and duration polling
- Buffering callbacks
- Error callbacks
- Seeking
- Resume position
- Player controls

Acceptance criteria:

- H.264 test media plays on the target Samsung TV.
- Playback position is reported correctly.
- Pause, resume, seek, and stop work.
- Unsupported media fails clearly rather than leaving a black screen.

### Phase 4: Plex playback planner

Deliverables:

- Plex media-part parsing
- Samsung capability table
- Direct-play decision
- Direct-stream decision
- Transcode URL generation
- Fallback after AVPlay failure
- Per-model diagnostics

Initial playback support should target H.264, AAC, AC-3, MP4, and HLS. Do not promise every HEVC, HDR, Dolby Vision, or subtitle profile until verified on physical TV models.

Acceptance criteria:

- Compatible media uses direct play when possible.
- Unsupported media falls back to Plex transcoding.
- The decision and fallback reason are visible in diagnostics.

### Phase 5: tracks and subtitles

Deliverables:

- Audio-track menu
- Subtitle menu
- Embedded text subtitles
- External subtitle support
- Subtitle delay if supported
- Track persistence
- Chapters
- Skip intro/credits when Plex metadata supports them

Acceptance criteria:

- At least one embedded subtitle and one external subtitle are verified on a physical TV.
- Track switching does not break playback.

### Phase 6: performance and release

Deliverables:

- Poster caching and request cancellation
- Memory limits
- Startup and first-frame measurements
- Focus-navigation measurements
- Rebuffering and AVPlay error telemetry
- Samsung model/firmware capability table
- CI package build
- Checksums and release artifacts
- Installation and troubleshooting documentation

## Tizen manifest

Start with a TV web application manifest containing:

- Tizen application ID and package ID
- Minimum supported Tizen profile selected from the physical test TV
- `index.html` entry point
- Network access
- TV input-device access
- File/storage access only if needed
- Screen-size feature
- A restrictive content-security policy compatible with Plex and playback URLs

The exact privilege set must be confirmed against the selected Tizen TV profile. Avoid broad permissions that are not required.

## Player abstraction

Use a platform-neutral interface so screens do not depend directly on AVPlay:

```ts
export interface Player {
  open(request: PlaybackRequest): Promise<void>;
  play(): void;
  pause(): void;
  stop(): Promise<void>;
  seek(seconds: number): void;
  getPosition(): number;
  getDuration(): number;
  selectAudioTrack(index: number): Promise<void>;
  selectSubtitleTrack(index: number): Promise<void>;
  setListener(listener: PlayerListener): void;
}
```

Implement:

- `AvplayPlayer`: production Samsung TV implementation
- `Html5Player`: browser/simulator fallback

The AVPlay adapter must enforce the documented AVPlay lifecycle states and reject invalid operations rather than relying on exceptions from the Samsung runtime.

## Remote input

Use standard `keydown` handling for arrows, Enter, and Back. Register only additional media keys that the app actually uses through the Tizen TV input-device API.

Required initial actions:

- Left
- Up
- Right
- Down
- Enter
- Back
- Play/pause if available
- Rewind/fast-forward if available

Map input into application actions rather than allowing screens to inspect raw key codes.

## Build scripts

### `package.json`

Add scripts similar to:

```json
{
  "scripts": {
    "tizen:build": "./scripts/tizen-build.sh",
    "tizen:package": "./scripts/tizen-package.sh",
    "tizen:install": "./scripts/tizen-install.sh",
    "tizen:run": "./scripts/tizen-run.sh",
    "tizen:uninstall": "./scripts/tizen-uninstall.sh",
    "tizen:logs": "./scripts/tizen-log.sh",
    "tizen:check": "./scripts/tizen-device-check.sh",
    "test:tizen": "vitest run tests/tizen"
  }
}
```

### Build script responsibilities

`scripts/tizen-build.sh` should:

1. Resolve the repository root.
2. Remove the previous build directory.
3. Bundle `tizen/app/src/main.ts` with TypeScript and esbuild.
4. Copy `index.html`, CSS, `config.xml`, icons, and static assets.
5. Validate that the output contains the manifest and entry point.
6. Write the output to a clearly named build directory.

### Package script responsibilities

`scripts/tizen-package.sh` should:

1. Require `TIZEN_CERT_PROFILE`.
2. Run the Tizen web build.
3. Invoke `tizen build-web`.
4. Invoke `tizen package -t wgt -s "$TIZEN_CERT_PROFILE"`.
5. Copy the resulting `.wgt` into `dist/tizen/`.
6. Print the package path and SHA-256 checksum.

Never commit certificate files, private keys, or passwords.

### Device scripts

Scripts should support:

```sh
TV_IP=192.168.1.50 ./scripts/tizen-device-check.sh
TV_IP=192.168.1.50 ./scripts/tizen-install.sh dist/tizen/<package>.wgt
TV_IP=192.168.1.50 ./scripts/tizen-run.sh
TV_IP=192.168.1.50 ./scripts/tizen-log.sh
TV_IP=192.168.1.50 ./scripts/tizen-uninstall.sh
```

The scripts should connect with SDB, fail on missing environment variables, and avoid embedding a TV address in source control.

## Installation methods

### Method 1: Samsung TV Developer Mode and SDB

Primary development method:

1. Install Tizen Studio.
2. Install Samsung TV Extensions, TV tools, and the certificate extension.
3. Enable Developer Mode on the Samsung TV.
4. Enter the development computer IP address on the TV.
5. Restart the TV if requested.
6. Create a Samsung certificate profile using the target TV’s DUID.
7. Connect using Device Manager or SDB.
8. Build and sign the `.wgt`.
9. Install with the Tizen CLI or IDE.
10. Launch and inspect logs with SDB/Web Inspector.

### Method 2: Tizen Studio GUI

Import `tizen/app` as a Tizen Web Project, select the Samsung certificate profile, connect a TV through Remote Device Manager, and run the project as a Tizen Web Application.

### Method 3: command-line local install

Use the shell scripts above for repeatable builds and installs. This is the preferred developer workflow after the first GUI-based certificate/device setup.

### Method 4: CI artifact build

CI should run host tests and produce an unsigned or development-signed package only when signing secrets are deliberately configured. Store certificate material and passwords in GitHub Actions secrets. Publish the `.wgt`, checksum, commit SHA, and supported profile metadata as build artifacts.

### Method 5: Samsung Seller Office

Publication is a later phase. It requires production signing, Samsung review/certification, screenshots, privacy disclosures, metadata, and supported-device information. Until publication, users must use Developer Mode installation.

## Plex authentication and security

Implement:

1. Request Plex PIN.
2. Display QR code/short URL.
3. Poll for completion.
4. Store the token in application storage.
5. Query Plex resources.
6. Select server.
7. Select Plex Home profile.
8. Clear tokens during logout/reset.

Never place Plex tokens in source, manifests, tests, screenshots, or logs. Add a development-only reset action for clearing local app state.

## Testing strategy

### Host tests

Test without a TV:

- Plex XML/JSON parsing
- Authentication state machine
- Playback-plan decisions
- Focus movement
- Search grouping
- Resume calculations
- Subtitle selection
- Route transitions

### Browser tests

Use Chromium with the HTML5 player fallback for:

- Login
- Home
- Library
- Search
- Details
- Player controls
- Keyboard remote simulation
- Network errors

### Emulator tests

Use the Samsung emulator for launch, navigation, input, storage, and network-error flows. Do not use emulator results as proof of physical-TV codec support.

### Physical-TV matrix

Record for every tested model:

- Model and year
- Tizen version and firmware
- Resolution
- Network type
- H.264 result
- HEVC result
- HDR result
- Audio result
- Subtitle result
- Direct-play result
- Transcode result
- AVPlay error/result

Initial media matrix:

- H.264/AAC MP4
- H.264/AC-3
- HEVC/AAC
- HEVC/E-AC-3
- 1080p
- 4K
- HDR10
- HLS
- Direct play
- Plex transcode
- Resume
- Seek
- Audio switching
- Subtitle switching

## First pull request scope

The first implementation PR should be intentionally small:

- Add Tizen web application shell.
- Add `config.xml`.
- Add TypeScript build and type declarations.
- Add placeholder 1920x1080 UI.
- Add Samsung remote input adapter.
- Add Tizen platform abstraction.
- Add AVPlay interface/types without claiming full playback support.
- Add build/package/install/run/uninstall/log scripts.
- Add Tizen development and installation documentation.
- Add CI host validation.
- Preserve the existing LG build unchanged.

After that PR is installed successfully on a physical Samsung TV, implement Plex authentication and browsing, then AVPlay playback.

## Explicit non-goals for the first port

Do not initially port:

- Starfish C/C++ bridge
- LG ACB video-plane code
- SDL2 renderer
- LG developer triggers
- LG-specific test harness
- webOS cross-compilation
- Full Rust-to-WebAssembly conversion
- Dolby Vision guarantees
- Native 60 FPS claims

These can be revisited after a working Tizen MVP exists.
