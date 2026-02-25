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

### Critical Hit Requirements

To perform a critical hit in 1.16.1, **all** conditions must be met simultaneously:

1. **Falling** — you must be in the downward arc of a jump (or falling off a block)
2. **Not on the ground** — cannot be touching floor, ladder, or vine
3. **Not in water** — swimming prevents crits (lava does not)
4. **No Blindness effect** — the status effect blocks crits
5. **Attack charge ≥ 84.8%** — the damage multiplier must be at least 0.848

Critical hits deal **1.5x damage** (applied after Strength effects, before enchantment damage).

!!! abstract "The 84.8% Threshold Explained"
    The wiki states crits require "84.8% attack cooldown." This number comes from the quadratic formula at 90% linear charge:

    \[
    0.2 + 0.8 \times (0.9)^2 = 0.2 + 0.648 = 0.848
    \]

    The game likely checks if linear charge \( p \geq 0.9 \), which produces 84.8% damage output. You **can** crit before 100% charge, but the 1.5x multiplier applies to the reduced damage — so a crit at 84.8% does less than a crit at 100%.

    | Scenario | Base Damage | × Crit | Total |
    |----------|-------------|--------|-------|
    | 100% charge, no crit | 1.00 | — | 1.00× |
    | 84.8% charge, crit | 0.848 | × 1.5 | 1.27× |
    | 100% charge, crit | 1.00 | × 1.5 | 1.50× |

    A rushed crit at 84.8% is still better than a non-crit at 100%, but a full-charge crit at 100% is **18% stronger** than a rushed crit.

### Sprint-Knockback vs Critical Hit

In Java 1.16.1, sprinting and critting are **mutually exclusive on the first hit**:

- If you're sprinting, your first hit triggers a sprint-knockback attack instead of a critical hit
- Sprint-knockback deals no extra damage — it only increases knockback
- **Exception**: After the first sprint-knockback, subsequent hits while holding sprint **can** be critical hits

For the dragon fight, this means: release sprint before your first hit if you want it to crit.

## Axe Damage in 1.16.1

| Axe | Base Damage | Attack Speed | Cooldown | Max Crit Damage |
|-----|------------|-------------|----------|-----------------|
| Wooden | 7 | 0.8 | 1.25s | 10.5 |
| Stone | 9 | 0.8 | 1.25s | 13.5 |
| Iron | 9 | 0.9 | 1.11s | 13.5 |
| Diamond | 9 | 1.0 | 1.00s | 13.5 |
| Netherite | 10 | 1.0 | 1.00s | 15.0 |

### Axe vs Sword for the Dragon

The Ender Dragon has **200 HP**. The tradeoff:

| Weapon | Damage/Hit | Cooldown | DPS (full charge) | Hits to Kill (crits) |
|--------|-----------|----------|-------------------|---------------------|
| Diamond Sword | 7 | 0.625s | 11.2 | 20 |
| Diamond Axe | 9 | 1.00s | 9.0 | 15 |
| Netherite Axe | 10 | 1.00s | 10.0 | 14 |

!!! tip "Speedrun Context"
    In a one-cycle bed strat, the axe is typically used for **finishing hits** between bed explosions, not as the primary damage source. Beds deal 100 damage each in the End. The axe matters most when you need 1-2 clean hits to finish the dragon after beds.

    With a diamond axe at full charge + crit: `9 × 1.5 = 13.5 damage`. Two crits = 27 damage — enough to close out after 6 bed explosions (approximately 600 raw damage, reduced by positioning variance).

## Optimal Dragon Fight Rhythm

When axing the dragon during a perch:

1. **Wait for full charge** — always. The quadratic penalty for early swings is severe.
2. **Jump-crit rhythm** — jump, hit the head on the way down, wait 1 full second, repeat
3. **Aim for the head** — headshots deal more damage than body hits
4. **Release sprint before the first hit** — otherwise you get sprint-knockback instead of a crit
5. **Position inside the bedrock rim** — reduces the chance of being flung by wings

The rhythm is: jump → swing on descent → land → wait for full bar → jump → swing → repeat.

---

*Damage formula and attack speed values sourced from [Minecraft Wiki — Damage](https://minecraft.wiki/w/Damage) and [Minecraft Wiki — Melee Attack](https://minecraft.wiki/w/Melee_attack). Critical hit mechanics from [Minecraft Wiki — Critical Hit](https://minecraft.wiki/w/Critical_hit). Axe stats from [Minecraft Wiki — Axe](https://minecraft.wiki/w/Axe). All values verified for Java Edition 1.16.1.*
