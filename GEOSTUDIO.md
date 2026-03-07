# GeoStudio & Playground - Visual Launchers for Binary Quad-Tree Grammar Engine

Two visual launchers for the Binary Quad-Tree Geometric Grammar Engine:
- **Playground.py** - Comprehensive showcase with all .geo scripts and descriptions
- **GeoStudio.py** - Export-focused launcher with preset browser

## Quick Start

```bash
# Launch the Playground Showcase (RECOMMENDED)
python Playground.py

# List all available .geo scripts
python Playground.py --list

# Run a specific .geo file directly
python Playground.py --geo examples/spiral.geo --depth 5 --speed 4

# Run a built-in demo
python Playground.py --demo conway_life --grid

# Launch GeoStudio (export-focused)
python GeoStudio.py
```

## Playground.py - Main Showcase

The Playground is the primary way to explore and run .geo scripts. It features:

- **Visual browser** - All .geo scripts organized by category
- **Live descriptions** - See what each script does before running
- **Auto grid detection** - Automatically enables grid mode for neighbor-aware scripts
- **Real-time controls** - Pause, resume, restart simulations
- **Export capabilities** - Save frames as PNG or export as GIF

### Playground Controls

| Action | Control |
|--------|---------|
| Load script | Click on script name in left panel |
| Pause/Resume | Click on simulation view |
| Toggle Pause | Space bar |
| Restart | R key |
| Export Frame | E key |
| Export GIF | G key |

### Command Line Options

```
--geo FILE       Run a specific .geo file
--demo NAME      Run a built-in demo by name
--list           List all available scripts
--grid           Force grid mode (auto-detected for most scripts)
--depth N        Recursion depth (default: 5)
--speed N        Ticks per second (default: 3.0)
```

## Available .geo Scripts

### Visual Demos

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Spiral` | Cycles through all four loop families in an 8-tick cycle | No |
| `Pulse Depth` | Pulses gate-on every 10 ticks, deep cells lock to Z-loop | No |
| `Stochastic` | Probabilistic rules for organic, non-deterministic evolution | No |
| `Depth Layers` | Different behaviors at different recursion depths | No |
| `Mask Set` | Specific mask values trigger special behavior | No |
| `Rotate Mirror` | Rotates and mirrors geometry using bit operations | No |

### Cell Variables

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Heat Spread` | Cells accumulate heat and change family at thresholds | Yes |
| `Composite` | Composite actions: SWITCH + EMIT + SET_VAR in one rule | Yes |

### Signal Propagation

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Signal Wave` | Signal-based communication between grid cells | Yes |
| `Forest Fire` | Trees catch fire from burning neighbors, burn out after 4 ticks | Yes |

### Cellular Automata

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Conway's Life` | Classic Game of Life with Birth/Survival rules | Yes |

### Neighbor Interactions

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Neighbor Spread` | Cells switch family when they detect specific neighbors | Yes |

### Multi-Program

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Vote Example` | Program-identity conditions with plurality voting | Yes |

### Dungeon Generation

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Dungeon Generator` | 6-phase procedural level generation | Yes |

### Ecosystem Simulation

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Ecosystem` | Predator/prey/plant simulation with emergent behavior | Yes |

### Basic Demo

| Script | Description | Grid Mode |
|--------|-------------|-----------|
| `Hello World` | Introduction to depth-based pattern switching | No |

## Detailed Script Descriptions

### ecosystem.geo - Predator/Prey/Plant Simulation

A three-tier ecosystem demonstrating emergent behavior:
- **Plants** (green/X_LOOP) - Grow, spread seeds, die of old age
- **Herbivores** (blue/Z_LOOP) - Eat plants, reproduce, flee from predators
- **Predators** (red/Y_LOOP) - Hunt herbivores, reproduce, die of starvation
- Uses cell variables (`age`, `energy`, `hunger`) and signals (`plant_food`, `prey_near`, `danger`)

**Run:**
```bash
python Playground.py --geo examples/ecosystem.geo
# or
python Playground.py --demo ecosystem --grid
```

### dungeon_generator.geo - Procedural Level Generator

Six-phase dungeon generation using cellular automata:
1. **Phase 1** (ticks 0-5) - Random noise seeding (45% walls)
2. **Phase 2** (ticks 6-15) - Cellular automaton smoothing
3. **Phase 3** (ticks 16-25) - Room expansion
4. **Phase 4** (ticks 26-40) - Corridor carving
5. **Phase 5** (ticks 41-55) - Door placement
6. **Phase 6** (ticks 56+) - Final polish

**Run:**
```bash
python Playground.py --geo examples/dungeon_generator.geo
```

### conway_life.geo - Conway's Game of Life

Classic cellular automaton with Birth/Survival rules:
- **Alive** = GATE_ON (1111 - white)
- **Dead** = GATE_OFF (0000 - dark)
- **Rules**: Survive with 2-3 neighbors, born with exactly 3 neighbors
- Uses 8-neighbor (Moore neighborhood) counting

**Run:**
```bash
python Playground.py --geo examples/conway_life.geo
# or
python Playground.py --demo conway_life --grid
```

### forest_fire.geo - Fire Spread Simulation

Recursive forest fire with signal propagation:
- **Trees** (green/X_LOOP) - Catch fire from burning neighbors
- **Fire** (red/Y_LOOP) - Emits fire signal, burns for 4 ticks
- **Burnt** (dark/GATE_OFF) - Empty space after fire burns out

**Run:**
```bash
python Playground.py --geo examples/forest_fire.geo
```

### heat_spread.geo - Cell Variables Demo

Cells accumulate "heat" and change family at thresholds:
- Y-loop cells increment heat each tick
- At heat >= 5, switch to X-loop and reset
- At heat >= 10 in X-loop, switch to Z-loop

**Run:**
```bash
python Playground.py --geo examples/heat_spread.geo
```

### signal_wave.geo - Inter-Cell Communication

Signal-based communication between grid cells:
- Y-loop cells emit "pulse" signal
- Z-loop cells receiving pulse switch to X-loop
- X-loop cells emit "ripple" signal
- Z-loop cells receiving ripple switch to DIAG_LOOP

**Run:**
```bash
python Playground.py --geo examples/signal_wave.geo
```

### spiral.geo - Loop Family Cycling

Cycles through all four loop families in an 8-tick cycle:
- tick%8=0 → Y_LOOP (red)
- tick%8=2 → X_LOOP (green)
- tick%8=4 → Z_LOOP (blue)
- tick%8=6 → DIAG_LOOP (gold)
- Deep cells (depth>=5) get sealed as GATE_ON

**Run:**
```bash
python Playground.py --geo examples/spiral.geo
```

### stochastic.geo - Probabilistic Evolution

Non-deterministic behavior using random conditions:
- 30% chance to GATE_ON each tick
- 50% chance for GATE to switch to Y_LOOP
- Deep cells randomly flip to different families

**Run:**
```bash
python Playground.py --geo examples/stochastic.geo
```

## Dependencies

```bash
pip install matplotlib pillow
```

## GeoStudio.py - Export Tool

GeoStudio provides additional export capabilities:

### Export Frames as PNG

```bash
python GeoStudio.py --export spiral --frames 60 --output my_frames/
```

### Export as Animated GIF

```bash
pip install Pillow
python GeoStudio.py --geo examples/spiral.geo --gif output.gif --frames 60
```

### GeoStudio Options

```
--geo GEO             Path to .geo file
--preset NAME         Run a preset by name
--depth N             Recursion depth (default: 5)
--speed N             Ticks per second (default: 3.0)
--export NAME         Export mode (geo file or preset name)
--frames N            Number of frames to export (default: 60)
--output PATH         Output directory (default: exports/)
--gif                 Export as animated GIF
--fps N               FPS for GIF export (default: 10)
--no-gui              Run without GUI
```

## Troubleshooting

### Simulation window doesn't appear
- Check if matplotlib is installed: `pip install matplotlib`
- Try running with `--list` to verify scripts are detected
- On Windows, ensure your display driver is up to date

### Geo script errors
- Use `--grid` flag for simulations that use neighbor conditions (`nb_any=`, `nb_count_gte=`, etc.)
- Cell variables (`var_heat`, `var_energy`, etc.) require grid mode to work properly
- Check syntax: use `tick_in=A..B` for ranges, `depth=0` for exact depth (not `depth==0`)

### Conway's Life not working
- Must use grid mode (auto-detected for conway_life.geo)
- Use lower depth (3-4) for better visibility
- The 8x8 default grid is small - patterns evolve quickly

### GIF export not working
- Install Pillow: `pip install pillow`
- GIF export requires re-running the simulation from the start
- Large frame counts may take a while to process

### Scripts not loading
- Ensure you're in the project directory: `cd d:\CodexTest\BinaryQuadTreeCPUTest`
- Check that .geo files exist in the `examples/` folder
- Run `python Playground.py --list` to verify all scripts are detected

## Quick Reference

### Most Popular Scripts

| Script | Command | Category |
|--------|---------|----------|
| Conway's Life | `python Playground.py --demo conway_life --grid` | Cellular Automata |
| Ecosystem | `python Playground.py --geo examples/ecosystem.geo` | Simulation |
| Dungeon | `python Playground.py --geo examples/dungeon_generator.geo` | Generation |
| Spiral | `python Playground.py --demo spiral` | Visual Demo |
| Forest Fire | `python Playground.py --geo examples/forest_fire.geo` | Signal Propagation |

### Keyboard Shortcuts (in GUI)

| Key | Action |
|-----|--------|
| Click script | Load and run |
| Click sim / Space | Pause/Resume |
| R | Restart |
| E | Export frame as PNG |
| G | Export as GIF |
