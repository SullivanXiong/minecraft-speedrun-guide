# Triangulation

Finding the stronghold efficiently is one of the biggest time-savers in a speedrun. This page covers the math behind eye throws, the boat eye technique for high-precision measurement, and an advanced optimization for capturing your boat angle in the nether.

## Standard Eye Throw

The basic approach: throw an Eye of Ender, note your angle with F3+C, move perpendicular, throw again, and triangulate from the two angles. Tools like [Ninjabrain Bot](https://github.com/Ninjabrain1/Ninjabrain-Bot) automate the math.

With two standard throws, the bot estimates stronghold position with a standard deviation of ~0.05-0.2 degrees per throw, depending on your FOV, resolution, and measurement skill.

## Boat Eye Technique

Boat eye is an advanced method developed by ExeRSolver and Sharpieman20 that achieves ~0.001 degree precision — accurate enough to find the stronghold with a **single eye throw**.

### How It Works

When you enter a boat in Minecraft, your player angle snaps to a quantized grid — multiples of 1.40625 degrees (for positive angles) or 0.140625 degrees (for negative angles). This snapping is deterministic and predictable.

Ninjabrain Bot exploits this:

1. You enter a boat and press F3+C — the bot records your snapped angle as a reference point
2. You exit the boat and throw an Eye of Ender, then press F3+C again
3. The bot knows your mouse sensitivity, so it knows the minimum angle increment your mouse can produce (`minInc`)
4. It reconstructs your exact angle: `boatAngle + round((eyeAngle - boatAngle) / minInc) × minInc`

By anchoring to the quantized boat angle, the bot resolves far more decimal places than F3+C alone provides.

### God Sensitivity

Not all mouse sensitivities are equal for boat eye. Minecraft's angle math uses floating-point arithmetic, and when angles wrap past 360 degrees (mod 360), certain sensitivity values introduce rounding errors that corrupt the measurement.

**"God sensitivities"** are values where this error stays minimal. They're [catalogued by ExeRSolver](https://gist.github.com/ExeRSolver/4a21753593d6f18d73c741e3de22e808) with a max-angle-before-error column. The [god-sens calculator](https://god-sens.torbaz.dev/) helps you find your optimal value.

!!! warning "A/D Keys Corrupt the Boat Angle"
    Pressing A or D while in a boat offsets your angle from the clean quantized grid. If you've pressed A/D in any boat during the run, you must **reset** by entering and exiting a boat without pressing A/D. This re-snaps your angle to the quantized value.

### Standard Boat Eye Workflow

```
1. Set sensitivity to a god-sens value (one-time setup)
2. Configure Ninjabrain Bot: Settings → High precision → Enable boat measurements
3. Enter your sensitivity from options.txt into the bot
4. In-game: press the bot's "boat" hotkey (enters MEASURING state)
5. Place and enter a boat (without pressing A/D)
6. Press F3+C → bot captures the quantized boat angle
7. Exit and break the boat
8. Throw Eye of Ender, press F3+C → bot triangulates with high precision
```

## Nether Portal Boat Reset (Advanced Optimization)

During nether travel, you'll likely press A/D while in boats (navigating bastions, traveling to portal coordinates). This corrupts your boat angle. The standard fix is to reset in the overworld by placing a boat, entering/exiting, and breaking it. But there's a faster way.

### The Optimization

Reset your boat angle **in the nether portal itself**, before transitioning to the overworld. The portal loading screen is dead time — you're waiting for the teleport anyway. By resetting there, you arrive in the overworld with a clean angle and can go straight into the eye throw.

**Standard workflow (slower):**

| Step | Location | Time |
|------|----------|------|
| Go through portal | Nether → Overworld | ~3s (loading) |
| Place boat | Overworld | ~0.5s |
| Enter/exit boat (reset) | Overworld | ~0.5s |
| F3+C (boat angle capture) | Overworld | ~0.3s |
| Break boat | Overworld | ~0.5s |
| Throw eye + F3+C | Overworld | ~1s |

**Optimized workflow (faster):**

| Step | Location | Time |
|------|----------|------|
| Place boat in portal | Nether | ~0.5s |
| Enter boat (reset) | Nether | ~0.2s |
| Press boat hotkey + F3+C | Nether | ~0.3s |
| Exit/break boat | Nether | ~0.5s |
| Go through portal | Nether → Overworld | ~3s (loading) |
| Throw eye + F3+C | Overworld | ~1s |

The boat placement, entry, capture, and cleanup happen during or before the portal transition instead of after it. Net savings: **~1-1.5 seconds** of overworld time eliminated.

### Why This Works — Source Code Verification

This technique is valid because **Minecraft's yaw (horizontal angle) is the same coordinate system in all dimensions**, and **Ninjabrain Bot's boat angle capture is dimension-agnostic**.

Verified in the [Ninjabrain Bot source code](https://github.com/Ninjabrain1/Ninjabrain-Bot):

**1. Boat angle capture ignores dimension.**
In [`PlayerPositionInputHandler.java:57-58`](https://github.com/Ninjabrain1/Ninjabrain-Bot/blob/main/src/main/java/ninjabrainbot/model/input/PlayerPositionInputHandler.java), the bot captures only `playerPosition.horizontalAngle()` — a raw float with no dimension info:

```java
if (preferences.usePreciseAngle.get()
    && dataState.boatDataState().enteringBoat().get())
    return new SetBoatAngleAction(
        dataState.boatDataState(),
        playerPosition.horizontalAngle(),  // no dimension check
        preferences);
```

The dimension check (`!playerPosition.isInOverworld()`) only gates eye throw creation at line 66 — not boat angle capture.

**2. The boat data model has no dimension field.**
[`BoatDataState`](https://github.com/Ninjabrain1/Ninjabrain-Bot/blob/main/src/main/java/ninjabrainbot/model/datastate/highprecision/BoatDataState.java) stores only: `boatAngle` (float), `enteringBoat` (boolean), `reducingModulo360` (boolean), `boatState` (enum). No dimension, no coordinates.

**3. The precision math is dimension-independent.**
[`BoatEnderEyeThrow.getPreciseBoatHorizontalAngle()`](https://github.com/Ninjabrain1/Ninjabrain-Bot/blob/main/src/main/java/ninjabrainbot/model/datastate/highprecision/BoatEnderEyeThrow.java) computes:

\[
\alpha' = \text{boatAngle} + \text{round}\left(\frac{\alpha - \text{boatAngle}}{\text{minInc}}\right) \times \text{minInc}
\]

This is pure modular arithmetic on angles. The quantization grid (multiples of 1.40625°) is identical in all dimensions.

**4. Triangulation coordinates come from the eye throw, not the boat.**
[`EnderEyeThrowFactory.java:22`](https://github.com/Ninjabrain1/Ninjabrain-Bot/blob/main/src/main/java/ninjabrainbot/model/datastate/endereye/EnderEyeThrowFactory.java) asserts `isInOverworld()` for eye throws. The x/z coordinates used for triangulation always come from the overworld F3+C, not from the nether boat capture.

**5. No reset on portal transition.**
The bot has no dimension-change handler for boat state. The stored boat angle persists until manually reset. This is correct behavior — the quantization grid anchor doesn't change across dimensions.

!!! abstract "Verdict: Confirmed Valid"
    Capturing the boat angle in the nether and using it for overworld eye throws produces identical results to capturing in the overworld. The bot's angle math is dimension-independent, and the positional data for triangulation comes entirely from the overworld eye throw.

---

*Boat eye technique developed by ExeRSolver and Sharpieman20. Source code analysis based on [Ninjabrain Bot](https://github.com/Ninjabrain1/Ninjabrain-Bot) (Java). God sensitivity values from [ExeRSolver's gist](https://gist.github.com/ExeRSolver/4a21753593d6f18d73c741e3de22e808). Nether portal optimization tip from SylveonMCSR.*
