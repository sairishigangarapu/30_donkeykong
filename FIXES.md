# Fixes Document

## Bug 1: Jump over barrel doesn't work

**Problem:** The scoring condition for jumping over a barrel used a narrow vertical window (`0 < barrel.pos.y - player.pos.y + BARREL_R < 40`) and a tight horizontal window (`< 12`). The hit-detection circle (radius 20 around the barrel center) overlapped the scoring window, so the player was usually registered as "hit" instead of "scored" when jumping over a barrel.

**Fix:** Replaced the broken `above` check with a clean `player_above` check: `player.pos.y < barrel.pos.y - BARREL_R` (player's feet above the barrel's top). The hit check now only fires when the player is NOT above the barrel, and the score check fires when the player IS airborne, above the barrel, and within 20px horizontally. This cleanly separates "hit" from "jumped over".

## Bug 2: RGB background based on score

**Problem:** `theme_color(score)` was a stub returning `None`.

**Fix:** Implemented using the standard number-to-RGB formula (24-bit integer bit extraction):
```python
return ((score >> 16) & 0xFF, (score >> 8) & 0xFF, score & 0xFF)
```
The score is treated as a 24-bit integer: bits 16-23 become red, bits 8-15 become green, bits 0-7 become blue. The background color shifts as the score increases.

## Bug 3: Multiplier for every barrel jumped

**Problem:** `on_barrel_jumped(player, barrel)` and `score_multiplier(score)` were stubs.

**Fix:**
- Added `self.barrels_jumped = 0` to `Player.reset()`.
- `on_barrel_jumped` increments `player.barrels_jumped` by 1 each time the player clears a barrel.
- `score_multiplier(player)` returns `player.barrels_jumped`, so the multiplier grows with every barrel jumped (1st barrel = 1x, 2nd = 2x, 3rd = 3x, etc.).
- Reordered the scoring block so `on_barrel_jumped` is called before `score_multiplier` is read, ensuring the count is current.

## Bug 4: 70% probability for barrels going down ramps

**Problem:** The ladder-detection window (`abs(self.pos.x - lx) < 3`) was too tight — on slower frames the barrel could skip past the 6px window entirely, making the 70% chance unreliable.

**Fix:** Widened the detection window from `< 3` to `< 6` so the barrel reliably triggers the probability check when passing a ladder. The probability remains `random.random() < 0.7` (70% chance to descend, 30% chance to keep rolling) as requested.
