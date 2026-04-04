# Testing 2-Player Cascade/Tie Scenario

## Scenario: 2 players (P0 and P1)

### Initial State:
- Current player: P0 (index 0)
- P0 throws 2 cook-offs
- Sets: G.pendingMatch = { count: 2, challengerIndex: 0 }
- Turn passes to P1 (index 1)

### P1's Turn:
- P1 rolls and gets 2 cook-offs (tie!)
- Code at line 1084: G.pendingMatch = { count: 4, challengerIndex: G.currentIndex }
  - G.currentIndex is 1 (P1)
  - So: G.pendingMatch = { count: 4, challengerIndex: 1 }
- Turn passes back to P0 (index 0)

### P0's Turn Again:
- P0 is now defending against challengerIndex=1 (P1)
- But P0 was the ORIGINAL challenger!

## The Bug:
In a tie cascade, the code sets `challengerIndex: G.currentIndex`, which makes the DEFENDER become the new challenger. This is wrong - the original challenger should remain the challenger, just with increased cubes.

In a 2-player game:
- P0 challenges P1 with 2
- P1 ties with 2
- Now it says P1 is challenging P0 with 4
- But the original challenge was P0 → P1

The cascade should go back to P1 (the original defender), not make P1 the challenger!
