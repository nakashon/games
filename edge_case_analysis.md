# Edge Case Analysis

## 1. Player with 0 cubes gets their turn
Lines 968-977 in beginTurn():
```javascript
if (p.cubes <= 0) {
  if (G.pendingMatch && G.pendingMatch.challengerIndex !== G.currentIndex) {
    // Skip this player, they've won but pending match carries on
    // Actually: they win! Pending cubes from gone chef are removed.
  }
  victory(G.currentIndex);
  return;
}
```
**Issue**: The comment says "Skip this player" but the code calls `victory()` regardless. The if-statement at line 971 does NOTHING - it just has a comment inside.

## 2. Win check during roll (lines 1127-1132)
```javascript
if (p.cubes <= 0) {
  p.cubes = 0;
  await wait(500);
  victory(G.currentIndex);
  return;
}
```
**Correct**: Player can win during their roll after plating/dumping.

## 3. Cascade in 2-player game (line 1084)
```javascript
G.pendingMatch = { count: total, challengerIndex: G.currentIndex };
```
**Potential Issue**: In a tie, the current player (defender) becomes the new challenger. In a 2-player game, this flips who is challenging whom. Let's trace:
- P0 challenges P1 with 2 cookoffs
- Turn moves to P1
- P1 rolls 2 cookoffs (tie)
- Line 1084: challengerIndex = G.currentIndex = 1 (P1)
- Turn moves to P0
- P0 must now defend against P1's challenge

This seems CORRECT for the "hot potato" mechanic - ties pass the challenge burden to the next player!

## 4. Die roll probability
Line 800: `const r = Math.random() * 6;`
- r < 3: Ticket (range 0-3) = 3/6 = 50% ❌ WRONG
- r < 4: Dump (range 3-4) = 1/6 = 16.67% ✓
- r < 5: Plate (range 4-5) = 1/6 = 16.67% ✓  
- else: Cookoff (range 5-6) = 1/6 = 16.67% ✓

**Expected** (from rules at line 498-511): 3/6 ticket, 1/6 dump, 1/6 plate, 1/6 cookoff
**Actual**: CORRECT! 3 faces, 1 face, 1 face, 1 face

## 5. Cook-off with dead challenger (lines 1058-1062)
```javascript
const challengerAlive = getPlayer(pending.challengerIndex).cubes >= 0;
if (challengerAlive) {
  getPlayer(pending.challengerIndex).cubes += total;
}
```
**Issue**: Uses `>= 0` which means 0 cubes = alive. But win condition is `<= 0`. So a player with 0 cubes is both "alive" here and should have won!

## 6. CSS .hidden class
Line 326: `.btn-roll.hidden { display: none; }`
Lines 990, 1006, 1135: Uses `.classList.add('hidden')` on btn-continue

**Issue**: `.btn-continue.hidden` CSS rule is NOT defined! Only `.btn-roll.hidden` exists.
