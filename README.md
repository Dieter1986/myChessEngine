# Chessturion — The 9×9 Edition

A chess variant played on a 9×9 board with three expansion pieces: the **Robot**, the **Spectre**, and the **Phoenix**.

This repository contains two self-contained HTML files — an interactive rulebook and a playable game. There is no build step, no package manager, and no dependencies.

---

## Contents

| File | What it is |
|---|---|
| `chessturion.html` | Interactive rulebook with a rotatable 3D board rendered on a plain 2D canvas. Five tabs: The Board, New Pieces, The Phoenix, Advanced Rules, Classic Rules. |
| `chessturion-game.html` | Playable game — you (White) versus a built-in bot (Black). Full rules engine, move log, and live piece-status panels. |

## Running it

Open either file directly in a browser:

```
open chessturion.html          # macOS
xdg-open chessturion.html      # Linux
start chessturion.html         # Windows
```

Both files work from the local filesystem (`file://`) — no server required. If you'd rather serve them:

```
python3 -m http.server 8000
# then visit http://localhost:8000/chessturion-game.html
```

Requires a modern browser with Canvas 2D support. No internet connection needed.

---

## The board

9 files (**a–i**) × 9 ranks (**1–9**). White's back rank is rank 1.

```
    a   b   c   d   e   f   g   h   i
  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
9 │ T │ P │ L │ D │ K │ L │ X │ P │ T │   Black back rank
8 │   │   │ S │   │   │   │ F │   │   │   Spectre (blue) · Phoenix (orange)
7 │ p │ p │ p │ p │ p │ p │ p │ p │ p │   Black pawns
6 │   │   │   │   │   │   │   │   │   │
5 │   │   │   │   │   │   │   │   │   │
4 │   │   │   │   │   │   │   │   │   │
3 │ p │ p │ p │ p │ p │ p │ p │ p │ p │   White pawns
2 │   │   │ S │   │   │   │ F │   │   │   Spectre (blue) · Phoenix (orange)
1 │ T │ P │ L │ D │ K │ L │ X │ P │ T │   White back rank
  └───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

| Symbol | Piece | Start squares | Value |
|---|---|---|---|
| `K` | King | e1 / e9 | — |
| `D` | Queen | d1 / d9 | 9 |
| `T` | Rook | a1, i1 / a9, i9 | 5 |
| `L` | Bishop | c1, f1 / c9, f9 | 3 |
| `P` | Knight | b1, h1 / b9, h9 | 3 |
| `X` | **Robot** | g1 / g9 | 3 |
| `S` | **Spectre** | c2 / c8 | 4 |
| `F` | **Phoenix** | g2 / g8 | 6 |
| `p` | Pawn | a3–i3 / a7–i7 | 1 |

The four starting squares of the Spectre and Phoenix are **Blessed Squares**, permanently colour-coded on the board (blue and orange) for the whole game.

---

## The expansion pieces

### 🤖 The Robot — `X`

Begins the game moving exactly like a King: one square in any direction.

**Absorption Protocol.** Every time the Robot captures an enemy piece, it *permanently* absorbs that piece's movement. Capture a Knight and it gains the L-leap forever. Capture a Rook and it also slides orthogonally. Absorbed abilities stack and are never lost — a Robot that has taken a Rook and a Bishop slides like a Queen.

**Robot–Phoenix Meltdown.** If a Robot captures a Phoenix by any means, **both pieces are destroyed instantly** and the Phoenix forfeits all remaining lives. The Robot cannot absorb the Phoenix. This is the only guaranteed way to remove a first-life Phoenix from the board.

### 👁 The Spectre — `S`

Glides 1 or 2 squares in any direction. The path must be clear — it cannot jump.

**Phase Shift.** May leave the board for exactly 3 turns. Returning it to any vacant square costs a full move. Fail to return it in time and it is permanently eliminated.

**Shadow Infiltration.** When capturing, the Spectre lands on the square *directly beyond* its victim in the same straight line. That landing square must be empty.

### 🔥 The Phoenix — `F`

Moves like a King: one square in any direction.

**Solar Countdown.** Survive **25 complete rounds** on its first life and its owner wins outright at the start of round 26. On the second or third life the threshold drops to **15 rounds**. Miss the claim and the countdown is forfeited permanently.

**Three Lives.** When captured, the Phoenix respawns on its home square (g2 for White, g8 for Black). Any piece standing there is destroyed on arrival — including your own. On the third capture it leaves the game for good.

**Fire Tribunal.** The nuclear option against an enemy Phoenix:

- Only available once the enemy Phoenix is on its **second or third life** — a first-life Phoenix is completely immune.
- Announced at the start of your turn; it consumes the entire turn.
- You permanently sacrifice your own pieces totalling **at least 8 points**. If that means the Queen, that is the price.
- The enemy Phoenix is removed from the game along with its countdown and remaining lives. Your own Phoenix is unaffected.

---

## Other rules

**Vanguard Pawns.** Any pawn that lands on an *enemy* Blessed Square is permanently upgraded and may thereafter move and capture backward as well as forward. The upgrade belongs to that specific pawn.

**Castling.** The King slides **three** squares toward a Rook, and the Rook jumps to the square the King crossed. Illegal from check, through check, or with any square between them occupied.

**Grand Promotion.** A pawn reaching the far rank may become a Queen, Rook, Bishop, Knight, Robot, or Spectre. It may **not** become a Phoenix — the Phoenix you start with is the only one you will ever have. A promoted Robot starts with an empty absorption list.

**No en passant.** The rule does not exist in Chessturion.

Everything else — check, checkmate, stalemate, the two-square pawn opening — follows standard FIDE chess.

---

## Playing against the bot

Click a piece to select it, then click a highlighted square to move. Special moves are highlighted distinctly.

- **New Game** — reset the board.
- **Phase Shift** — remove your Spectre from the board. To bring it back, drag it from the side panel onto any vacant square.
- **Declare Fire Tribunal** — appears only when you are eligible. Pick sacrifices totalling ≥8 points and confirm.

The side panels track each Phoenix's remaining lives and Solar Countdown, each Spectre's off-board timer, and — for both sides — exactly which movement types each Robot has absorbed so far.

---

## Implementation notes

Both files are single-document HTML: markup, styles, and script inline, no external requests.

**Rules engine** (`chessturion-game.html`) is a conventional make/unmake design. `pseudoMoves()` generates candidate moves per piece type, `legalMoves()` filters out anything that leaves the King in check, and `applyMoveToBoard()` returns a fresh board rather than mutating in place.

**Robot state** lives on the piece itself as an `absorbed` array of piece-type characters. Because it is stored on the piece and deep-copied along with the board, absorbed abilities propagate correctly through the bot's lookahead without any separate bookkeeping.

**Bot** is minimax with alpha–beta pruning at depth 1, plus a material and piece-square evaluation that credits the Robot for what it has absorbed and rewards an advancing Solar Countdown. A deliberate 28% blunder rate — picking a random legal move instead of the best one — brings it to roughly 650 ELO. To make it stronger, raise the search depth and lower the blunder rate in `botMove()`.

**3D board** (`chessturion.html`) is hand-rolled software rendering on a 2D canvas: a small vector math helper, painter's-algorithm depth sorting, and Staunton-style pieces built from shaded cylinders, disks and spheres. Drag to rotate, scroll to zoom, hover a square for its coordinate and contents. Touch gestures work too.

---

## License

No license specified yet — add one before sharing publicly.
