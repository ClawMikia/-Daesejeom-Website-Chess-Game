# 대세점 (Daesejeom)

A cinematic, browser-based chess arena with customizable boards, piece avatars, CPU opponents across multiple difficulty levels, full audio, and a zoomable board.

Built as standalone HTML with **zero dependencies and no build step** — every feature below ships in the committed files.

---

## Quick Start

There is nothing to install, compile, or serve.

```
git clone <repo-url>
cd -Daesejeom-Website-Chess-Game
```

Open **`index.html`** in any modern browser. For the best experience use a local server instead of `file://` so audio and `sessionStorage` behave predictably:

```bash
python -m http.server 8000    # then visit http://localhost:8000/index.html
```

---

## Two Play Modes

The app ships as both a single page and a multi-page flow. Both contain the identical game engine; only the surrounding shell differs.

### Single-page mode — `index.html`

Everything lives on one long scrolling page: hero, setup panel, and live board. Click **Forge Your Army** to jump to the setup panel, then **Start playing against the CPU** to begin in place. The hover inspector tooltip and the verdict modal are available here.

### Multi-page mode — the remaining five pages

A classic routed flow with a top navbar:

| Page | Purpose |
| --- | --- |
| `home.html` | Landing page — hero, feature overview, mode switch |
| `setup.html` | Army configuration; writes the chosen setup to `sessionStorage` |
| `play.html` | The game board; auto-starts a match when a saved setup exists |
| `how-to-play.html` | Rules reference and walkthrough |
| `about.html` | Project background and credits |

Setup choices are handed between pages under the **`chessSetup`** key in `sessionStorage` (`saveSetupToStorage()` in `setup.html`, `loadSetupFromStorage()` in `play.html`). Because it is `sessionStorage` rather than `localStorage`, the handoff survives page-to-page navigation but clears when the tab is closed.

### Ghost shells

`about.html`, `how-to-play.html`, and `setup.html` keep hidden `#sidebar`, `#boardShell`, and `#boardGridSurface` elements purely so the shared game script finds the DOM nodes it expects. They are marked `display: none` and are not interactive. This is why the sound panel and zoom controls are injected only where their host container is actually displayed.

---

## Features

### Customization

- **25 Board Skins** — `Board1.png` … `Board25.png`
- **6 Piece Types × 4 Variants** — independent avatar choice for King, Queen, Rook, Bishop, Knight, and Pawn
- **15 Display Colors** — blue, red, yellow, green, orange, purple, brown, cyan, magenta, pink, lime, flesh, gray, white, black
- **4 CPU Difficulty Levels**:

  | Tier | Depth | Behaviour |
  | --- | --- | --- |
  | Gae-teol (개털) — Easy | 0 | Random legal moves with slight capture preference |
  | Aegyo-sal (애교살) — Normal | 1 | Short tactical greed with randomness among strong options |
  | Chok Chok (촉촉) — Hard | 2 | Alpha-beta search two plies deep |
  | Areumdaun (아름다운) — Expert | 3 | Deeper alpha-beta with stronger move ordering |

The CPU rolls its **own** random avatar variants and a colour different from the player's on every new match, so the two armies are always visually distinct.

### Chess engine

- Full legal move generation for all piece types
- Check, checkmate, and stalemate detection
- Castling, kingside and queenside, with rights tracking
- En passant, including pawn double-step
- Pawn promotion to queen
- Algebraic move notation in the move log
- Captured-piece tracking, split by capturer
- Per-match timer

The AI combines material and piece-square table evaluation with mobility, alpha-beta pruning, and MVV-LVA-style move ordering. Every candidate move is evaluated on a deep clone of the game, so the real board is never mutated during search.

### Audio

Looping background music plus four sound effects, driven by a small audio engine (`AUDIO_SOURCES`, `audioState`) shared by all pages.

| File | Trigger | Volume |
| --- | --- | --- |
| `music/background.mp3` | Loops continuously (`loop = true`) | 0.30 |
| `sounds/navigate.mp3` | Pointer hover / keyboard focus on any interactive UI | 0.32 |
| `sounds/select.mp3` | Clicking or selecting any button or clickable element; picking up a piece | 0.50 |
| `sounds/place.mp3` | A piece being placed or moved to a new square | 0.60 |
| `sounds/capture.mp3` | A piece capturing an opponent, by player **or** CPU | 0.66 |

Implementation notes:

- **Autoplay compliance.** Browsers block audio until a user gesture, so music starts on the first `pointerdown` or `keydown` and self-heals if that first attempt is rejected.
- **Overlap pool.** Each effect keeps a pool of 4 `Audio` instances, so rapid events (hovering across board squares, fast moves) never cut each other off.
- **Navigation throttle.** Hover is throttled to 130 ms per element, so moving the pointer within a tile does not retrigger the sound.
- **Move sounds are funneled** through a single call at the top of `makeLiveMove()`, the one function both the player and the CPU pass through — so no move can be missed. Capture is detected from `move.captured || move.enPassantCapture`, which means en passant also plays the capture sound.
- **Toggles.** A Sound panel exposes independent music and effects switches — full switch rows in the sidebar on `index.html`/`home.html`, compact `Music` / `SFX` chips in the top navbar on the pages that hide the sidebar. The panel picks whichever host is actually visible.

### Board zoom

A `− / percentage / +` control zooms the board from **50% to 200%** in 10% steps.

The board scales by changing `.board-shell`'s **width**, not by `transform: scale()`. This matters: the move animation derives ghost-piece positions from `getBoundingClientRect()` and positions them in unscaled pixels inside the shell. Under a transform those measurements come back multiplied by the zoom factor and every sliding piece would land on the wrong square. Scaling width keeps the shell's local coordinate system 1:1 with CSS pixels, so the animation and tooltip positioning needed no changes.

The shell sits inside a `overflow: auto` viewport, so zooming past the panel gives scrollbars instead of the board overlapping the timer and controls. The base width is measured from the viewport's *parent*, which keeps the appearing scrollbar from feeding back into the measurement and causing drift. Zoom lives in `state.view` rather than `state.game`, so it survives Rematch and Reset.

Controls are only built on pages where the board is displayed.

### Verdict modal

When the game reaches a terminal state, a modal appears with the result, split from the engine's status string into a headline and a detail line — `Checkmate. You win the duel.` renders as **Checkmate** over *You win the duel.* The badge is coloured by outcome: green for a player win, red for a CPU win, gold for a draw.

Two actions are offered:

- **Rematch** — re-randomizes the CPU and starts a fresh match
- **Back to main** — returns to the hero section on single-page/multi-page home, or navigates to `home.html` on the routed pages

The modal is dismissible by backdrop click or `Escape`, so the final board stays reviewable and the existing Rematch/Reset controls are never locked out. Focus moves to the Rematch button on open.

### Hover inspector

On `index.html` and `play.html`, hovering or focusing a square shows a tooltip with the coordinate, square colour, piece name and variant, ownership, legal-move count, and current status (selected, check, last move, captured).

It is `position: absolute` and scales with the board via `zoom: var(--bubble-zoom)`, which is inherited — so the border, padding, type scale, and tail arrow all resize together from a single property. Sizing still uses `offsetWidth`/`offsetHeight`, which report the layout box and already ignore zoom, so placement math stayed consistent.

### Board auto-fit

Board artwork does not all share the same playable area, so `estimateBoardFit()` samples the loaded image on a canvas, scores candidate offsets for checkerboard parity, cell adjacency, and alpha coverage, then caches the result in `boardFitCache`. The 8×8 grid is positioned over the detected board in percentages, which keeps it correct at any zoom level and across all 25 skins.

### Responsiveness

Desktop uses a fixed sidebar with live match summary, status, and timer. Below 980px it collapses into a hamburger drawer with a full-screen overlay. Below 720px card radii and paddings tighten further.

---

## Project Structure

```
├── index.html          # Single-page mode (hero + setup + live board)
├── home.html           # Multi-page landing
├── setup.html          # Army configuration
├── play.html           # Dedicated game board
├── how-to-play.html    # Rules reference
├── about.html          # About page
├── README.md
├── .gitignore
├── music/
│   └── background.mp3  # Looping background music
├── sounds/
│   ├── navigate.mp3    # UI hover / focus
│   ├── select.mp3      # UI click / selection
│   ├── place.mp3       # Piece placed or moved
│   └── capture.mp3     # Piece captured
└── Image Assets/
    ├── app_icon.png
    ├── background.png
    ├── Boards/         # 25 board skins (Board1.png … Board25.png)
    └── Pieces/
        ├── King/       # 4 variants (King4 is .webp)
        ├── Queen/      # 4 variants
        ├── Rook/       # 4 variants
        ├── Bishop/     # 4 variants
        ├── Knight/     # 4 variants (note: knight4.png is lowercase)
        └── Pawn/       # 4 variants
```

Roughly 3,500–4,200 lines per page. Each page embeds its own copy of the full stylesheet and game script, so every page is independently openable.

---

## How to Play

1. **Configure** — pick a board skin, one avatar variant per piece type, your display colour, and a difficulty. The summary panel updates live.
2. **Start the duel** — the CPU rolls its own variants and colour, then the starting position loads on your chosen board.
3. **Play** — click one of your pieces to reveal legal targets, then click a destination. The CPU replies automatically.
4. **Inspect** — hover any piece for its details; zoom in or out to suit your screen.
5. **Finish** — on checkmate or stalemate the verdict modal offers an immediate rematch or a return to the main view.

---

## Chess Rules Implemented

- Standard movement for all six piece types
- Check and checkmate detection
- Stalemate detection
- Castling (kingside and queenside) with castling-rights tracking
- En passant captures
- Pawn promotion to queen
- Algebraic move notation
- Captured-piece tracking
- Match timer

---

## Browser Support

Any modern browser. The app uses standard DOM APIs, CSS custom properties, `sessionStorage`, and the Web Audio API.

Note: `zoom` on the hover tooltip requires a reasonably current browser (Chrome/Edge/Safari always; Firefox 126+).

---

## Notes for Contributors

- **No build step.** Edit the HTML files directly and reload. There is no bundler, transpiler, or package manifest.
- **The game script is duplicated per page.** A change to shared logic must be applied to all six files. Every shared block sits at a consistent 4-space indent so the same edit can be applied mechanically — this is how the audio, zoom, and verdict features were rolled out.
- **Guard optional DOM lookups.** Some pages intentionally omit elements such as `#squareBubble`. Any code path that touches them must check for existence first, or `initializeUI()` will throw and abort the rest of page setup.
- **Do not modify the audio files.** They are reference assets used as-is.

---

## License

MIT License
