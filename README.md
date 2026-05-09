# 🟡 PAC-MAN Clone

> **⚠️ Work in Progress** — This is an actively developed Pac-Man clone built as a single HTML file using HTML5 Canvas. Features are being added and refined incrementally.

![Status](https://img.shields.io/badge/status-work%20in%20progress-yellow)
![Tech](https://img.shields.io/badge/tech-HTML5%20Canvas-blue)
![Libraries](https://img.shields.io/badge/libraries-none-green)

---

## 🎮 Play

Open `index.html` in any modern browser — no build step, no dependencies.

```
# Or serve locally:
npx -y http-server . -p 8080 -o
```

### Controls

| Key | Action |
|---|---|
| `↑` `↓` `←` `→` | Move Pac-Man |
| `W` `A` `S` `D` | Move Pac-Man (alt) |

---

## ✅ Implemented Features

### Step 1 — Grid & Maze
- 21×21 tile grid rendered on HTML5 Canvas (20px tiles)
- Maze defined as an editable 2D array (`1` = wall, `0` = path)
- Classic layout with border walls, inner blocks, and a ghost house
- Blue walls with depth shading

### Step 2 — Pac-Man Movement
- Pac-Man placed at a starting tile with arrow key / WASD input
- Direction buffering for responsive turning
- Wall collision prevention
- Animated yellow arc with opening/closing mouth and eye

### Step 3 — Pellets & Scoring
- Every path tile filled with a pellet dot on init
- Pellets removed on contact (+10 points each)
- Live score counter and persistent high score (localStorage)
- Win condition when all pellets are eaten

### Step 4 — Ghost
- Single red ghost starts at the centre of the ghost house
- Classic ghost shape with dome top and wavy bottom
- Directional eyes that follow movement
- Game Over on collision with Pac-Man
- Restart button on win/lose overlay

### Step 5 — Smart Ghost & Power Pellets
- BFS pathfinding toward Pac-Man (replaces random movement)
- 4 power pellets at maze corners (pulsing animation)
- Frightened mode: ghost turns blue, runs away from Pac-Man
- Flashing warning before frightened mode expires
- Eating a frightened ghost scores 200 points and resets it to the ghost house

---

## 🚧 Known Limitations / TODO

- [ ] Only one ghost — classic Pac-Man has four with unique behaviors
- [ ] No fruit bonus items
- [ ] No multi-level progression (speed increase, maze changes)
- [ ] No lives system (currently 1 life per game)
- [ ] No sound effects or music
- [ ] Tunnel wrapping only on row 9
- [ ] Ghost house exit logic could be improved
- [ ] Mobile touch controls not implemented

---

## 🏗️ Tech Stack

- **Single HTML file** — no external libraries or frameworks
- **HTML5 Canvas** for all rendering
- **`requestAnimationFrame`** for the game loop
- **Google Fonts** — Press Start 2P (retro arcade font)
- **localStorage** — high score persistence

---

## 📁 Project Structure

```
Pac-Man_clone/
├── index.html        # Complete game (HTML + CSS + JS)
├── README.md         # This file
├── debug/
│   └── fix.md        # Bug tracking & fix log
└── tests/
    └── tests.html    # Automated logic test suite (78 tests)
```

---

## 📝 License

Personal project — feel free to learn from and build upon it.
