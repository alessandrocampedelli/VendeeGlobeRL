# Vendée Globe RL — Simulation Episode Viewer

An interactive web-based viewer for visualizing simulation episodes produced by a Reinforcement Learning agent trained to complete the Vendée Globe solo sailing race.

---

## Project Overview

### What is the Vendée Globe?

The Vendée Globe is a solo, non-stop, unassisted around-the-world sailing race departing from Les Sables-d'Olonne, France. The route covers approximately 24,000 nautical miles, passing through the Atlantic Ocean, rounding the Cape of Good Hope, crossing the Southern Ocean past Leeuwin and Cape Horn, and returning to the start.

### The RL Agent

The core of the project is a **Proximal Policy Optimization (PPO)** agent built with [Stable-Baselines3](https://stable-baselines3.readthedocs.io/), trained inside a custom [Gymnasium](https://gymnasium.farama.org/) environment called `VendeeGlobeEnv`.

The environment simulates the physical dynamics of a sailing boat navigating a realistic wind field. At each timestep, the agent observes its position, heading, speed, distance to the next waypoint, distance from the coast, and local wind conditions — then decides which heading to steer.

**Key design decisions:**

- **Wind data**: Real ERA5 reanalysis wind fields from the Copernicus Climate Data Store, decoded with `cfgrib`/`eccodes`. Three historical seasons were used for training (2016–17 and 2020–21), with the 2024–25 season held out as an unseen test set.
- **Reward function (V2)**: A simplified reward based on progress toward the next waypoint, a universal coast-proximity penalty, and a bonus for finishing. Hardcoded geographic penalties (Cape Horn, Africa) were deliberately removed so the agent could learn to navigate these passages autonomously — a cleaner narrative for evaluation.
- **Network architecture**: Three fully connected layers of 256 units each (`net_arch = [256, 256, 256]`), trained for approximately 10 million timesteps.
- **Hyperparameters**: `n_steps = 4096`, `batch_size = 512`, `ent_coef = 0.02`, `learning_rate = 2e-4`, `clip_range = 0.2`, `target_kl = 0.02`.
- **Integration step**: `dt_hours = 1.0` (one simulated hour per environment step).

### Training vs. Test

The agent was trained on two seasons and evaluated on the **2024–25 season**, which it had never seen during training. This split tests true out-of-distribution generalization: does the agent navigate correctly with wind patterns it was not trained on?

### Simulation Results

After training, 50 deterministic episodes were run on the test season. Each episode runs the trained policy from the same starting position (Les Sables-d'Olonne) through up to 5,000 steps. Observed outcome distribution:

| Outcome | Frequency | Description |
|---|---|---|
| ✅ Finish | ~80% | Agent completes all 7 waypoints in ~84–85 days |
| ✕ DNF (early) | ~11% | Agent stops before reaching WP1 (mid-Atlantic) |
| ✕ DNF (late) | ~9% | Agent passes Cape Horn but fails to complete the return leg |

The best finishing time observed was **84.5 days** at an average speed of **12.4 knots**.

---

## Technical Stack

| Component | Tool |
|---|---|
| RL framework | Stable-Baselines3 (PPO) |
| Environment | Custom Gymnasium env (`VendeeGlobeEnv`) |
| Wind data | ERA5 reanalysis via Copernicus CDS (`cdsapi`, `cfgrib`) |
| Visualization | Plotly.js (orthographic Scattergeo globe) |
| Language | Python 3.11, JavaScript (vanilla) |
| IDE | Google Antigravity (VS Code fork) |
| Python environment | `venv311` with `ipykernel` Jupyter kernel |