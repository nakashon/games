# Final Bug Report

## CRITICAL BUGS

### 1. Dead Code in beginTurn() (Lines 968-977)
**File:** index.html:968-977
**Severity:** Medium (confusing but not breaking)
**Problem:** The if-statement at line 971 contains only a comment and does nothing
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
The inner if-statement (line 971) has no code - just comments. It should either handle the pending match or be removed.

### 2. Dead Challenger Check is Wrong (Lines 1058-1062)
**File:** index.html:1058
**Severity:** High (gameplay bug)
**Problem:** Uses `>= 0` to check if challenger is alive, but win condition is `<= 0`
```javascript
const challengerAlive = getPlayer(pending.challengerIndex).cubes >= 0;
```
**Issue:** A player with EXACTLY 0 cubes should have won (per line 968 and 1127), but this code treats them as "alive" and gives them more cubes!

**Scenario:**
1. Player A has 2 cubes, challenges Player B with 2 cook-offs
2. Player A now has 0 cubes (should win next turn!)
3. Player B rolls 1 cook-off (loses)
4. Line 1058: `cubes >= 0` → true (Player A is "alive")
5. Line 1060: Player A gets +3 cubes (2+1)
6. Player A doesn't win!

**Fix:** Should be `> 0` not `>= 0`

### 3. Potential Race Condition in AI Auto-Continue (Lines 992-996, 1138-1139)
**File:** index.html:992-996, 1138-1139
**Severity:** Medium
**Problem:** Two setTimeout calls for AI behavior, one at beginning of turn (line 995) and one after roll (line 1137)

At line 995 (in beginTurn):
```javascript
setTimeout(() => doRoll(), 800 + Math.random() * 600);
```

At line 1137 (end of doRoll):
```javascript
if (p.isAI) {
  setTimeout(() => nextTurn(), 1000);
}
```

**Scenario where this could break:**
- If `doRoll()` is already running when the user clicks "NEXT CHEF" button
- `isAnimating` flag at line 1000 prevents this, but...
- At line 1136, `isAnimating = false` is set BEFORE the setTimeout
- So if nextTurn() is called externally right after line 1136, it could start a new turn while the AI setTimeout is still pending

**Actual Check:** Line 1142 has `if (isAnimating) return;` which protects against this. So this is OK!

## EDGE CASES - ALL HANDLED CORRECTLY

### Player with 0 cubes gets turn
✓ Line 968 checks at start of turn and declares victory

### 2-player cascade/tie
✓ Line 1084 correctly makes the defender the new challenger (hot potato mechanic)
✓ Log message at line 1106 correctly shows cascade to next player

### Die roll probability  
✓ Correct: 3/6 ticket, 1/6 dump, 1/6 plate, 1/6 cookoff

### AI auto-continue
✓ Protected by isAnimating flag
✓ nextTurn() checks isAnimating before proceeding

## MINOR ISSUES

### CSS: Missing .btn-continue.hidden rule
**VERIFIED AS FIXED:** Line 333 defines `.btn-continue.hidden { display: none; }`
✓ No issue here!

## SUMMARY
- 1 High severity bug: Dead challenger check uses >= instead of >
- 1 Medium severity code quality issue: Dead code in beginTurn()
- Everything else works correctly!
