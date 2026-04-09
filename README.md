# neural-ca

Neural-controller cellular automata experiments — single-file browser simulations in which a small neural network learns to tune the underlying CA rules in real time, steering the system toward sustained complex behavior.

These are not implementations of any known named CA rule. The grid dynamics use a hodgepodge-style update (healthy → infected → ill → healthy), but the parameter tuning layer is original: a tiny MLP predicts how rule changes will affect system behavior, and two cooperating RL agents (explorer + exploiter) use that model to pick actions that maximize a composite **complexity** signal. A smoothed **boredom** score tracks long flat stretches in live / change / complexity; it **shapes the tabular value update** (mild penalty) and **biases exploration** toward novelty when things get too repetitive — steering toward sustained interesting dynamics without forcing constant instability.

## Experiments

### [hodgepodge_trend_aware.html](hodgepodge_trend_aware.html)

**Trend-aware agents with 2-step lookahead.**

The controller tracks short-term trends in live-cell fraction, change rate, and complexity, and uses them to preempt bad attractors before they fully develop — a *flee mode* that fires when trends indicate the system is headed for death, saturation, or freeze. Two agents alternate: an **explorer** that values novelty (visiting uncharted regions of the live/change space), and an **exploiter** that mines known high-value regions. Both use a 2-step lookahead over the learned dynamics model.

Key mechanics:
- **Tiny MLP (15→16→4):** trained online; inputs include behavior, parameters, action one-hot, and boredom features; predicts Δlive, Δchange, Δedges, Δsvar
- **Value function:** tabular over an 8×8 (live, change) archive; updated with temporal-difference learning
- **Flee mode:** trend-based early warning — fires before the system actually reaches a danger threshold
- **2-step lookahead:** scores each action by simulating one follow-up action using the learned model
- **Symmetry options:** 8-fold, 4-fold, or none — affects initial condition and exploration space
- **Archive panel:** shows the learned value function; color = estimated long-term complexity

### [hodgepodge_fields_and_hits.html](hodgepodge_fields_and_hits.html)

**Spatial parameter fields + greatest-hits snapshot memory.**

Instead of uniform global rule parameters, each of the three rule knobs (g, k1, k2) is a spatially varying field — a 6×6 control grid bilinearly interpolated across the full 144×144 simulation grid. This means the rule is different in different parts of the space, enabling richer pattern coexistence. Two additional continuous knobs (`growthExp`, `diagWeight`) further shape the update dynamics. When the system reaches a high-complexity state it has not seen recently, a snapshot is saved to a "greatest hits" library (up to 6). When the system gets stuck, rather than randomizing completely, it preferentially restores one of these snapshots — resuming from a previously good configuration.

Key mechanics:
- **Spatial fields:** 6×6 grids for g, k1, k2 — bilinearly interpolated to full resolution; visualized in real time
- **Extended action space:** 12 actions — shift field mean up/down, bump a random location in the g field, smooth all fields, adjust growthExp or diagWeight
- **Beam search lookahead (width 3, depth 3):** replaces the 2-step lookahead; explores action sequences rather than single actions
- **Greatest-hits memory:** up to 6 snapshots of high-complexity states, including full field state; restored on stuck detection with 70% probability
- **Larger NN (23→20→4):** wider input (includes boredom features) for the richer parameter state

## Live demos

Runnable copies on [novaspivack.com](https://www.novaspivack.com) (WordPress may append `-1`, `-2`, … to filenames when the base name already exists in Media):

- **Trend-aware agents:** [hodgepodge_trend_aware-4.html](https://www.novaspivack.com/wp-content/uploads/2026/04/hodgepodge_trend_aware-4.html)
- **Spatial fields + greatest hits:** [hodgepodge_fields_and_hits-4.html](https://www.novaspivack.com/wp-content/uploads/2026/04/hodgepodge_fields_and_hits-4.html)

## Blog (essays + galleries)

Write-ups that **cross-reference each other** and go deeper on motivation and UI:

- [Neural CA: Trend-Aware Agents Learn to Keep a Cellular Automaton Alive](https://www.novaspivack.com/science/neural-ca-trend-aware-agents)
- [Neural CA: Spatial Parameter Fields and Greatest-Hits Memory](https://www.novaspivack.com/science/neural-ca-spatial-fields-greatest-hits)

## How to run

Open either `.html` file directly in a browser. No server, no install, no dependencies. Everything runs in-page.

Click **Run / pause** to start. The simulation begins tuning immediately once the neural network has seen enough data (~30–40 controller steps). The **auto** checkbox enables/disables the controller; uncheck it to explore parameter space manually.

## Controls

| Control | Effect |
|---------|--------|
| Run / pause | Start or stop simulation |
| Step | Advance one tick |
| Reset | Re-initialize grid and learning state |
| Forget | Reset only the learning state (keep grid) |
| Symmetry | Initial condition and placement symmetry |
| Seed | Initial grid pattern |
| Skip (frame skip) | Ticks rendered per animation frame — higher = faster but choppier |
| auto | Enable/disable neural controller |

## What to watch for

- The **archive panel** (bottom right) shows learned value estimates across the (live, change) space — watch it develop structure over time
- The **trace panel** (bottom) shows rule parameters and complexity over the last 240 ticks
- In the fields experiment, the **field panels** (right side) show the current spatial distribution of each rule parameter — bumps and gradients appear as the controller explores
- The status line shows the current controller action: `explorer g↑`, `FLEE-death k1↑`, `RESTORE hit`, etc.

## Related

- [LACE](https://github.com/novaspivack/lace) — a different CA extension: topology-aware rules where cell connections affect state
- [SocioLife](https://github.com/novaspivack/sociolife) — agent-based artificial life simulation
- [novaspivack.com/research](https://www.novaspivack.com/research) — formal research program

## License

Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)  
© Nova Spivack — [novaspivack.com](https://www.novaspivack.com)
