# Dragon Fight

Optimizing damage output against the Ender Dragon requires understanding the attack cooldown system. The damage formula is non-linear, and small timing mistakes cost more than you'd expect.

## Attack Cooldown Mechanics (Java 1.16.1)

### The Damage Formula

After swinging a weapon, damage recovers on a **quadratic** curve, not a linear one:

\[
\text{Damage multiplier} = 0.2 + 0.8p^2
\]

Where \( p \) is the linear charge completion:

\[
p = \text{clamp}\left(\frac{t + 0.5}{T},\ 0,\ 1\right)
\]

- \( t \) = ticks since last attack
- \( T = \frac{20}{\text{attack speed}} \) (in ticks)

This means damage recovery is **back-loaded**. The first 50% of the wait gives you very little; the last 10% matters a lot.

| Linear Charge (p) | Damage Multiplier | % of Max |
|-------------------|-------------------|----------|
| 0% | 0.200 | 20% |
| 25% | 0.250 | 25% |
| 50% | 0.400 | 40% |
| 75% | 0.650 | 65% |
| 85% | 0.778 | 77.8% |
| 90% | 0.848 | 84.8% |
| 100% | 1.000 | 100% |

!!! warning "The 85% Trap"
    When the axe visually "pops up" on screen (~85% linear charge), it **looks** ready. But you're only dealing 77.8% damage — a 22.2% loss per hit. Over 10 hits on the dragon, that's 2+ extra hits you need to land.

### The Attack Indicator Bar

The fill level of the attack indicator bar represents the **damage proportion**, not the time proportion. This is confirmed directly by the [Minecraft Wiki](https://minecraft.wiki/w/Damage):

> "The fill-level of the attack indicator bar (as a proportion of its total width) is exactly equal to the damage the held weapon would deal if it were used now, in proportion to its maximum damage."

So when the bar looks 85% full, you're dealing 85% damage. The bar itself is accurate — the trap is swinging when it's "almost" full.

!!! note "Visual Lag"
    The indicator display uses `clamp(t / T, 0, 1)` while actual damage uses `clamp((t + 0.5) / T, 0, 1)`. The display lags by half a game tick. At 20 TPS, this is a 25ms discrepancy — negligible in practice but technically present.

## Axe Damage in 1.16.1

| Axe | Base Damage | Attack Speed | Cooldown |
|-----|------------|-------------|----------|
| Wooden | 7 | 0.8 | 1.25s |
| Stone | 9 | 0.8 | 1.25s |
| Iron | 9 | 0.9 | 1.11s |
| Diamond | 9 | 1.0 | 1.00s |
| Netherite | 10 | 1.0 | 1.00s |

### Axe vs Sword for the Dragon

The Ender Dragon has **200 HP** and is **immune to critical hits**. This changes the weapon calculus compared to normal mobs — you cannot get the 1.5x crit multiplier, so raw DPS at full charge is all that matters.

| Weapon | Damage/Hit | Cooldown | DPS (full charge) | Hits to Kill |
|--------|-----------|----------|-------------------|-------------|
| Diamond Sword | 7 | 0.625s | 11.2 | 29 |
| Diamond Axe | 9 | 1.00s | 9.0 | 23 |
| Netherite Axe | 10 | 1.00s | 10.0 | 20 |

The sword has higher DPS because its faster attack speed (1.6 vs 1.0) more than compensates for its lower per-hit damage. Against the dragon specifically, where crits don't apply, the sword's faster cooldown is a clear advantage for sustained melee.

!!! tip "Speedrun Context"
    In a one-cycle bed strat, the axe is typically used for **finishing hits** between bed explosions, not as the primary damage source. Beds deal up to 100 damage each in the End. The weapon matters most when you need 1-2 clean hits to finish the dragon after beds.

    With a diamond axe at full charge: 9 damage. Two full-charge hits = 18 damage — enough to close out after bed explosions have done the heavy lifting.

## Optimal Dragon Fight Rhythm

When hitting the dragon during a perch:

1. **Wait for full charge** — always. The quadratic penalty for early swings is severe.
2. **Aim for the head** — headshots deal more damage than body hits
3. **Position inside the bedrock rim** — reduces the chance of being flung by wings
4. **Consistent timing** — swing, wait for the bar to fill completely, swing again. Don't rush.

The rhythm is: swing → wait for full bar → swing → repeat. With a diamond axe, that's one swing per second.

---

*Damage formula and attack speed values sourced from [Minecraft Wiki — Damage](https://minecraft.wiki/w/Damage) and [Minecraft Wiki — Melee Attack](https://minecraft.wiki/w/Melee_attack). Axe stats from [Minecraft Wiki — Axe](https://minecraft.wiki/w/Axe). All values verified for Java Edition 1.16.1.*
