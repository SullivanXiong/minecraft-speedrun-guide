# Crafting Decisions

Not everything you *can* craft is worth the time. This page analyzes which tools and items are worth crafting during a speedrun, based on how much time they save versus how much time they cost.

## Stone Hoe for Hay Bales

!!! abstract "Verdict: Worth It"
    Crafting a stone hoe costs ~1.5 seconds and saves **3.35 seconds** when mining 7 hay bales. Net gain: **~1.85 seconds**.

### The Debate

A common claim in the speedrun community is that crafting a stone hoe in a village run is a waste of time. The reasoning: you already have faster tools, hay bales break quickly enough, and the crafting time isn't worth it.

The math disagrees.

### Mining Speed Comparison

| Method | Time per Hay Bale | 7 Hay Bales | Source |
|--------|-------------------|-------------|--------|
| Bare hands | 0.75s | 5.25s | Minecraft Wiki |
| Stone hoe | 0.20s | 1.40s | Minecraft Wiki |

Time saved by using a stone hoe on 7 hay bales:

\[
\Delta t = 7 \times (0.75 - 0.20) = 7 \times 0.55 = 3.85 \text{ seconds}
\]

### Crafting Cost

To craft a stone hoe, you need 2 cobblestone and 2 sticks (1 plank → 2 sticks, so effectively 1 plank + 2 cobblestone).

Assuming you already have a pickaxe (standard in any village run):

| Action | Time |
|--------|------|
| Mine 2 cobblestone (iron pickaxe) | ~1.0s (0.5s per block) |
| Open crafting table + craft hoe | ~0.5s |
| **Total crafting cost** | **~1.5s** |

### Net Time Saved

\[
\text{Net} = 3.85 - 1.5 = 2.35 \text{ seconds}
\]

!!! note "Conservative Estimate"
    Even if we add 0.5 seconds for inventory management and assume only 7 hay bales (a low-average village), the net savings are still **~1.85 seconds**. With larger hay bale counts, the savings scale linearly.

### Scaling Analysis

The break-even point — where crafting time equals time saved — is remarkably low:

\[
n \times 0.55 = 1.5 \implies n = \frac{1.5}{0.55} \approx 2.73 \text{ hay bales}
\]

You only need to mine **3 hay bales** for the stone hoe to pay for itself. Any village with hay bales will have at least this many.

| Hay Bales | Bare Hands | Stone Hoe | Net Saved |
|-----------|-----------|-----------|-----------|
| 3 | 2.25s | 0.60s | 0.15s |
| 5 | 3.75s | 1.00s | 1.25s |
| 7 | 5.25s | 1.40s | 2.35s |
| 10 | 7.50s | 2.00s | 4.00s |
| 14 | 10.50s | 2.80s | 6.20s |

### When to Skip the Hoe

There are situations where the hoe is *not* worth it:

- **You only need 1-2 hay bales** — Below the break-even point
- **No cobblestone available** — Mining stone specifically for this isn't worth it
- **You already have a better hoe** — Iron or diamond hoe from village chests (rare but possible)
- **Tick-perfect splits** — If you're routing a specific seed and can avoid hay bales entirely

### Conclusion

For the vast majority of village runs, crafting a stone hoe is a free 2+ seconds. The break-even is only 3 hay bales, and most villages have 7-14. The community perception that it's "not worth it" is mathematically incorrect.

---

*Mining speed data sourced from [Minecraft Wiki - Breaking](https://minecraft.wiki/w/Breaking). Crafting times estimated from in-game testing.*
