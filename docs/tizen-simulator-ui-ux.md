# Samsung Tizen Simulator UI/UX Specification

## Visual target

The simulator should feel like a clean, dark streaming application designed for viewing from several feet away:

- 16:9 TV layout
- 1920×1080 default composition
- Large text and controls
- Large artwork cards
- High-contrast focus state
- Smooth horizontal shelves
- Minimal small controls
- No desktop-style forms
- No visible browser chrome inside the TV viewport

## Layout principles

- Reserve safe margins around the screen edges.
- Keep navigation and primary actions reachable with directional input.
- Never require a pointer for a primary flow.
- Use one visible focus target at a time.
- Preserve focus when returning from a detail, search, or player route.
- Use skeleton placeholders rather than layout shifts while artwork loads.
- Keep player controls hidden until input is received or playback is paused.

## Screen wireframes

### Shell

```text
┌──────────────────────────────────────────────────────────────┐
│  PLXNATIVE TIZEN                         [Diagnostics: OFF]  │
│                                                              │
│                         Tizen Shell                         │
│                                                              │
│                   Press Enter to continue                   │
│                                                              │
│  ↑ ↓ ← → Navigate       Enter Select       Back Return      │
└──────────────────────────────────────────────────────────────┘
```

### Login

```text
┌──────────────────────────────────────────────────────────────┐
│  Sign in to Plex                                             │
│                                                              │
│       ┌──────────────┐        Scan with your phone           │
│       │    QR CODE   │        or visit: plex.tv/link        │
│       └──────────────┘        Code: ABCD                     │
│                                 Waiting for approval...      │
│                                                              │
│                         [Cancel]                             │
└──────────────────────────────────────────────────────────────┘
```

### Home

```text
┌──────────────────────────────────────────────────────────────┐
│  PlxNative       Home   Movies   Shows   Search   Settings   │
│                                                              │
│  Continue Watching                                           │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │
│  │poster│ │poster│ │poster│ │poster│ │poster│               │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘               │
│                                                              │
│  Recently Added                                             │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │
│  │poster│ │poster│ │poster│ │poster│ │poster│               │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘               │
│                                                              │
│  Focused item: Example Movie                                 │
└──────────────────────────────────────────────────────────────┘
```

### Library

```text
┌──────────────────────────────────────────────────────────────┐
│  Movies                                      Sort   Filter    │
│                                                              │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │
│  │poster│ │poster│ │poster│ │poster│ │poster│               │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘               │
│                                                              │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │
│  │poster│ │poster│ │poster│ │poster│ │poster│               │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘               │
│                                                              │
│  A  B  C  D  E  F  G  H  I  J  K  L  M  N  O  P ...        │
└──────────────────────────────────────────────────────────────┘
```

### Details

```text
┌──────────────────────────────────────────────────────────────┐
│  ┌──────────────┐       Example Movie                       │
│  │    POSTER    │       2024 · 2h 04m · PG-13                │
│  └──────────────┘       A short description...               │
│                                                              │
│                           [ Play ]                            │
│                           [ Resume ]                          │
│                           [ Trailer ]                         │
│                                                              │
│  Cast: Actor One   Actor Two   Actor Three                    │
└──────────────────────────────────────────────────────────────┘
```

### Search

```text
┌──────────────────────────────────────────────────────────────┐
│  Search                                                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ star wars_                                              │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Movies  [poster] Star Wars                                 │
│  Shows   [poster] Star Wars: Example Series                 │
│  People  [person] Example Actor                             │
└──────────────────────────────────────────────────────────────┘
```

### Player

```text
┌──────────────────────────────────────────────────────────────┐
│                         VIDEO                                │
│                                                              │
│  Example Movie                                               │
│  ━━━━━━━━━━━━━━━●──────────────────────────────              │
│  00:42:18                                      02:04:10      │
│                                                              │
│  ▶  ❚❚       −10s    +10s       Audio       Subtitles       │
└──────────────────────────────────────────────────────────────┘
```

### Diagnostics

```text
┌──────────────────────────────────────────────────────────────┐
│  Diagnostics                                                 │
│  Platform: Browser simulator                                 │
│  Target profile: Samsung TV 2024                             │
│  Player: HTML5 fallback                                      │
│  Route: player                                               │
│  Focus: play-button                                          │
│  FPS: 59.8                                                    │
│  Network: 42 ms                                               │
│  Plex server: Mock PMS                                        │
│  Playback mode: Direct play                                  │
│                                                              │
│  [Toggle slow network] [Toggle image delay] [Reset app]      │
└──────────────────────────────────────────────────────────────┘
```

### Virtual remote

```text
              ┌──────────────┐
              │   POWER      │
              │      ↑       │
              │   ←  OK  →   │
              │      ↓       │
              │     BACK     │
              │  PLAY PAUSE  │
              │  REW   FF    │
              └──────────────┘
```

## Focus behavior

- Focus must be visible at all times outside the video-only player state.
- Focus must move predictably by spatial direction, not DOM order alone.
- A focused card may scale slightly, brighten, or receive a border, but must not cause neighboring layout shifts.
- Focus should return to the previously focused item when a route is revisited.
- Pressing Back should close an overlay before leaving the current screen.
- Focus transitions should be testable independently from rendering.

## Motion and performance

- Avoid expensive full-screen DOM updates during navigation.
- Prefer transforms and opacity for focus animation.
- Cancel artwork requests when cards leave the useful lookahead window.
- Use controlled skeleton animation so stress mode can measure its cost.
- Support 30 FPS and 60 FPS simulator profiles for regression testing.
