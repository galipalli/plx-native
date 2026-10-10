# Samsung Tizen Simulator Specification

## Purpose

Define a desktop-browser simulator for the Samsung Tizen version of PlxNative. The simulator must resemble a 1920×1080 Samsung TV application rather than a developer dashboard and must allow keyboard-driven testing before deploying to a physical TV.

## Scope

The simulator covers:

- TV-sized rendering at 1920×1080
- Optional 1280×720 and 3840×2160 profiles
- Keyboard input mapped to Samsung remote actions
- Screen routing and focus management
- Mock Plex data for credential-free UI development
- HTML5 video playback as the browser fallback for Samsung AVPlay
- Login, home, library, detail, search, player, settings, and diagnostics flows
- Network, artwork, authentication, playback, and performance fault simulation

## Simulator window

The default viewport is a fixed 1920×1080 TV surface. Browser chrome must not appear inside the simulated TV surface.

The simulator should provide:

- Dark TV-style visual treatment
- Large readable typography
- Large poster cards
- Strong focus indication
- No mouse dependency for core navigation
- Optional virtual remote beside the viewport
- A diagnostics indicator that is disabled by default

## Keyboard mapping

| Keyboard input | TV action |
|---|---|
| Arrow keys | Navigate focus |
| Enter | Select/focused action |
| Escape or Backspace | Samsung Back |
| Space | Play/pause |
| `R` | Rewind |
| `F` | Fast-forward |
| `I` | Toggle diagnostics |

Keyboard and virtual-remote input must dispatch the same application actions. Screens must not contain separate navigation implementations for each input device.

## Initial shell acceptance criteria

The initial shell must prove that:

- The app starts in the simulator.
- The viewport fills the configured TV surface.
- Keyboard input works.
- Focus is visible and moves correctly.
- Enter activates the focused control.
- Back returns to the previous route.
- Routing works.
- The simulator can later swap browser adapters for Tizen APIs.

## Screen requirements

### Shell

Show the PlxNative Tizen title, a simple Tizen Shell message, an Enter-to-continue action, and the current keyboard hints.

### Login

Show a QR-code/short-link login layout with a sample code and waiting status. Development mode must offer fake login and local mock-account selection so navigation never requires real Plex credentials.

### Home

Provide top-level navigation for Home, Movies, Shows, Search, and Settings. Display shelves such as Continue Watching and Recently Added. Focus moves horizontally between cards and vertically between shelves. Focused cards should enlarge slightly or receive a bright border. Artwork loading must support skeleton placeholders and configurable delays.

### Library

Provide a poster grid with Sort and Filter actions, scrolling, unwatched-only mode, alphabetical navigation, placeholder artwork, and large-library support. Stress mode must be able to generate 500–2,000 fake movies.

### Details

Show poster, title, year, duration, rating, description, Play, Resume, Trailer, and cast/person navigation. Test data must support missing artwork and incomplete metadata.

### Search

Support physical keyboard typing and an optional on-screen TV keyboard. Results must be grouped by Movies, Shows, and People. Test empty results, debounce behavior, slow requests, failed requests, and grouped results.

### Player

Use the browser HTML5 `<video>` element in the simulator. The physical Tizen target will use Samsung AVPlay through the common player interface.

The player must expose:

- Play/pause
- Seek
- Resume
- Buffering state
- Playback errors
- Audio-track menu
- Subtitle menu
- Chapters
- Skip intro/credits
- End-of-video behavior
- Automatic Up Next

### Settings and diagnostics

A development-only diagnostics panel must be reachable with `I` or from Settings. It should report platform, target profile, player adapter, route, focused element, FPS, simulated memory, network latency, Plex source, and playback mode.

Diagnostics must not be visible in release packages unless explicitly enabled.

## Virtual remote

The optional virtual remote should contain directional buttons, OK, Back, Play/Pause, Rewind, and Fast-forward. It must dispatch the same action names as keyboard input.

## Non-goals

The simulator does not need to reproduce the existing LG renderer exactly. The first goal is to validate Tizen launch, navigation, authentication, browsing, and playback flows. Visual parity and advanced GPU optimization can follow after the Tizen MVP is functional.
