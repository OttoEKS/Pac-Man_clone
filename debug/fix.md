# 🐛 Debug & Fix Log

All bugs discovered during development and their solutions, in chronological order.

---

## Bug #1 — Game Speed Too Fast (Initial)

**Reported:** 2026-05-05  
**Severity:** 🟡 Medium — Gameplay  
**Status:** ✅ Fixed

### Problem
Both Pac-Man and the ghost moved far too quickly, making the game unplayable. The ghost with BFS pathfinding felt especially fast since it takes the optimal route.

### Root Cause
Frame-based movement delays were set too low for 60fps:
- Pac-Man: `moveDelay = 8` → ~7.5 tiles/sec
- Ghost: `moveDelay = 10` → ~6 tiles/sec

### Fix (Attempt 1 — Partial)
Increased delays moderately:
```diff
- moveDelay: 8    // Pac-Man
+ moveDelay: 10   // ~6 tiles/sec

- moveDelay: 10   // Ghost
+ moveDelay: 15   // ~4 tiles/sec
```

**Result:** Still too fast. Needed a bigger adjustment.

### Fix (Attempt 2 — Final)
Doubled the original values to match classic Pac-Man pacing:
```diff
- moveDelay: 10   // Pac-Man
+ moveDelay: 16   // ~3.75 tiles/sec

- moveDelay: 15   // Ghost
+ moveDelay: 20   // ~3 tiles/sec
```

### Lesson Learned
With BFS pathfinding the ghost doesn't waste any moves, so it needs to be considerably slower than Pac-Man to feel fair. Raw speed difference matters more when movement is optimal.

---

## Bug #2 — Ghost Stuttering / Lagging in Frightened Mode

**Reported:** 2026-05-05  
**Severity:** 🟡 Medium — Visual  
**Status:** ✅ Fixed

### Problem
When Pac-Man ate a power pellet and the ghost entered frightened mode, the ghost visibly stuttered and felt "laggy" — it would barely move.

### Root Cause
The frightened mode added too many extra delay frames on top of the already-slowed ghost:
```javascript
// Ghost base delay was 15, bonus was +6 = 21 frames between moves
// At 60fps, ghost moved only ~2.85 tiles/sec — too slow to even appear animated
const delay = ghost.frightened ? ghost.moveDelay + 6 : ghost.moveDelay;
```

### Fix
Reduced the frightened speed penalty from `+6` to `+3`:
```diff
- const delay = ghost.frightened ? ghost.moveDelay + 6 : ghost.moveDelay;
+ const delay = ghost.frightened ? ghost.moveDelay + 3 : ghost.moveDelay;
```

Final speeds: Normal = 20 frames (~3 tiles/sec), Frightened = 23 frames (~2.6 tiles/sec). The ghost is noticeably slower when frightened but still moves smoothly.

---

## Bug #3 — Ghost Teleporting When Frightened Mode Ends

**Reported:** 2026-05-05  
**Severity:** 🔴 High — Game-breaking  
**Status:** ✅ Fixed

### Problem
When the ghost transitioned from frightened (blue) back to normal (red), it would instantly jump/teleport to a distant tile instead of continuing from its current position.

### Root Cause
The BFS `ghost.path` array was calculated **before** frightened mode started and was never cleared. When frightened mode ended:

1. Ghost was now at a completely different position (it had been running away)
2. The old `ghost.path` still contained tiles leading to where Pac-Man **used to be**
3. The code did `ghost.row = next.r; ghost.col = next.c` without checking adjacency
4. The ghost would jump to a stale path tile far from its actual position

```javascript
// OLD CODE — no validation, just blindly follows stale path
if (ghost.path.length > 0) {
    const next = ghost.path.shift();
    ghost.row = next.r;   // Could be 10+ tiles away!
    ghost.col = next.c;
}
```

### Fix (Three-layer protection)

**Layer 1 — Clear path on mode enter:**
```diff
  ghost.frightened = true;
+ ghost.path = [];
+ ghost.pathTimer = 0;
  frightenedTimer = 480;
```

**Layer 2 — Clear path on mode exit:**
```diff
  if (frightenedTimer <= 0) {
      ghost.frightened = false;
+     ghost.path = [];
+     ghost.pathTimer = 0;
  }
```

**Layer 3 — Adjacency validation before following any path step:**
```javascript
if (ghost.path.length > 0) {
    const next = ghost.path[0];
    const dr = Math.abs(next.r - ghost.row);
    const dc = Math.abs(next.c - ghost.col);
    const isAdjacent = (dr + dc === 1) || (dr === 0 && dc === COLS - 1);
    
    if (isAdjacent) {
        ghost.path.shift();
        ghost.row = next.r;
        ghost.col = next.c;
    } else {
        // Path is stale — force fresh BFS
        ghost.path = bfs(ghost.row, ghost.col, pacman.row, pacman.col);
        ghost.pathTimer = 0;
    }
}
```

### Lesson Learned
Any cached pathfinding data must be invalidated whenever the agent's mode or position changes significantly. Always validate path steps before applying them — never trust stale data.

---

## Summary Table

| # | Bug | Severity | Category | Status |
|---|---|---|---|---|
| 1 | Game too fast | 🟡 Medium | Gameplay | ✅ Fixed |
| 2 | Ghost lag in frightened mode | 🟡 Medium | Visual | ✅ Fixed |
| 3 | Ghost teleporting on mode change | 🔴 High | Game-breaking | ✅ Fixed |
