# Samsung Tizen Simulator Test Plan

## Simulator modes

### Shell mode

Tests startup, routing, focus movement, remote input, Back behavior, and error screens without Plex credentials.

### Mock Plex mode

Uses generated local data:

- Fake account
- Fake servers
- Fake libraries
- Fake posters
- Fake metadata
- Fake search results

This is the default mode for UI development and automated screen tests.

### Real Plex mode

Uses an explicitly configured Plex account and server for authentication, library data, metadata, and playback URLs. It must be opt-in and must never commit or print credentials.

### Playback mode

Tests local MP4 files, HLS streams, Plex playback URLs, resume, seeking, audio menus, and subtitle menus through the browser HTML5 fallback.

### Stress mode

Generates 2,000 library items, 20 shelves, delayed artwork, rapid navigation, and long scrolling sessions. It is intended to expose focus, layout, memory, and rendering regressions before deployment to a TV.

## Fault-injection controls

The simulator should be able to force:

- Normal playback
- Buffering
- Network failure
- Unsupported codec
- Transcode fallback
- End of stream
- AVPlay-like error
- Slow network
- Offline mode
- Plex server unavailable
- Delayed artwork
- Missing artwork
- Low-memory mode
- Authentication expiration
- Subtitle failure

These controls should be available only in development builds or behind an explicit simulator flag.

## Test layers

### Host tests

Run without a TV:

- Plex XML/JSON parsing
- Authentication state machine
- Playback-plan decisions
- Focus movement
- Search grouping
- Resume calculations
- Subtitle selection
- Route transitions

### Browser tests

Run in a desktop browser with the HTML5 player:

- Login screen
- Home screen
- Library screen
- Search screen
- Detail screen
- Player controls
- Keyboard remote simulation
- Network errors
- Fault-injection controls

### Tizen emulator tests

Use the Samsung emulator for:

- App launch
- Navigation
- Input
- Storage
- Network-error flows
- Basic route transitions

Emulator playback is not proof of physical-TV codec support.

### Physical Samsung TV tests

Record the following for every model and firmware:

- Model and year
- Tizen version
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

## Platform boundary

The simulator and the physical Tizen app should share screens, routes, Plex models, focus engine, and the platform-neutral player interface.

| Simulator | Samsung TV |
|---|---|
| HTML5 player | Samsung AVPlay |
| Browser localStorage | Tizen storage APIs |
| Keyboard events | TV remote events |
| Browser console | SDB/Web Inspector logs |
| Mock Plex server | Real Plex server |
| HTML/CSS renderer | Tizen web runtime |

Only platform adapters should differ between the simulator and the physical TV target.

## Release-readiness checks

Before calling the simulator ready for physical-TV deployment:

1. Shell mode passes keyboard and Back-navigation tests.
2. Mock Plex mode browses home, libraries, details, and search.
3. Stress mode completes long navigation without focus loss or unbounded memory growth.
4. Browser playback completes play, pause, seek, resume, and stop scenarios.
5. Fault injection produces visible, recoverable errors.
6. Diagnostics can identify the active adapter and route.
7. Real Plex mode is opt-in and secrets are absent from source and logs.
8. The same route and player tests pass when the Tizen adapter replaces the browser adapter.
