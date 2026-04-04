# Tracing "Dead Challenger" Scenario

## Scenario Setup:
- Player A has 2 cubes
- Player B has 5 cubes
- Player A's turn

## Turn 1 (Player A):
1. Player A rolls 2 cubes
2. Gets 2 cook-offs
3. Line 1044: `p.cubes -= counts.cookoffs` → A now has 0 cubes
4. Line 1094: Sets `G.pendingMatch = { count: 2, challengerIndex: 0 }`
5. Line 1127: Check `if (p.cubes <= 0)` → TRUE
6. Line 1130: `victory(G.currentIndex)` → Player A WINS immediately!

## Wait... Player A wins DURING their roll, not on next turn!

Let me re-read lines 1041-1048:
```javascript
// Cook-offs leave the player's hand
if (counts.cookoffs > 0) {
  p.cubes -= counts.cookoffs;
}

renderStations();
await wait(400);

// ─── Resolve Cook-Off ───
```

Then lines 1127-1132:
```javascript
// ─── Check Win ───
if (p.cubes <= 0) {
  p.cubes = 0;
  await wait(500);
  victory(G.currentIndex);
  return;
}
```

## So the issue is:
Player A with 2 cubes throws 2 cook-offs → goes to 0 → WINS before B can even respond!

This seems... wrong? The cook-off should resolve first!

## Actually, re-reading more carefully:

The win check at line 1127 happens AFTER the cook-off resolution (lines 1050-1113).

So:
1. A throws 2 cook-offs, goes to 0 cubes
2. Cook-off resolves
3. If A loses, A gets all the cubes back
4. THEN win check at line 1127

## So the "dead challenger" code at line 1058 runs when:
- A previously challenged B
- B is now rolling (defender)
- B wins the cook-off
- Need to give cubes to A (the challenger)
- But what if A reached 0 cubes somehow between challenging and now?

## Can A reach 0 cubes after challenging but before B defends?

NO! Because:
- A challenges on their turn
- Turn immediately passes to B
- B must defend
- There's no opportunity for A to lose cubes in between

## So when would `challengerAlive` be false?

The only way is if A had NEGATIVE cubes, which shouldn't happen.

Actually, looking at the code, cubes CAN go negative! Look at line 1128:
```javascript
p.cubes = 0;  // Clamp to 0
```

But before that line, p.cubes could be -3 if they plated 5 cubes from 2!

Wait no, you can't plate more than you have. You roll YOUR cubes. So if you have 2 cubes, you roll 2 dice.

## Conclusion:
The `>= 0` check is defensive programming but doesn't actually fix a real bug because:
- Cubes can't go negative through normal gameplay
- A challenger can't reach 0 cubes between challenge and defense
- The check should still be `> 0` for correctness
