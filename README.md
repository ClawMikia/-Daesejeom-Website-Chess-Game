# 대세점 (Daesejeom)

A cinematic, browser-based chess arena with customizable boards, piece avatars, and CPU opponents with multiple difficulty levels.

## Features

- **25 Board Skins** - Multiple chess board designs to choose from
- **6 Piece Types × 4 Variants** - Independent avatar choices for King, Queen, Rook, Bishop, Knight, and Pawn
- **15 Display Colors** - Customize accent colors for player pieces
- **4 CPU Difficulty Levels**:
  - Gae-teol (개털) - Easy: Random legal moves with slight capture preference
  - Aegyo-sal (애교살) - Normal: Short tactical greed with randomness among strong options
  - Chok Chok (촉촉) - Hard: Alpha-beta search two plies deep
  - Areumdaun (아름다운) - Expert: Deeper alpha-beta search with stronger move ordering
- **Full Chess Rules**: Legal move generation, check detection, castling, en passant, queen promotion
- **Responsive Design**: Desktop sidebar navigation with mobile hamburger menu
- **Standalone HTML**: No external dependencies - runs offline in any modern browser

## Project Structure

```
├── home.html          # Main landing page
├── setup.html         # Army configuration page
├── play.html          # Game board page
├── about.html         # About page
├── how-to-play.html   # Instructions page
├── index.html         # Single-page mode
├── .gitignore         # Git ignore rules
└── Image Assets/
    ├── app_icon.png   # Application icon
    ├── background.png # Background image
    ├── Boards/        # 25 chess board designs
    │   ├── Board1.png - Board25.png
    └── Pieces/        # Chess piece avatars
        ├── King/
        ├── Queen/
        ├── Rook/
        ├── Bishop/
        ├── Knight/
        └── Pawn/
```

## How to Play

1. **Configure** - Select a board frame, choose one avatar variant for each chess piece type, and pick your display color and difficulty
2. **Start the duel** - The CPU rolls its own random avatar variants and a different accent color, then the initial chess position loads on the chosen board
3. **Play on the board** - Click one of your pieces, review the legal targets, and click a destination square. The CPU answers automatically after your move

## Game Rules Implemented

- Standard chess movement rules
- Check and checkmate detection
- Stalemate detection
- Castling (kingside and queenside)
- En passant captures
- Pawn promotion to queen
- Move logging with algebraic notation
- Piece capture tracking
- Game timer

## Supported Browsers

Any modern browser with JavaScript support (Chrome, Firefox, Safari, Edge, etc.)

## License

MIT License