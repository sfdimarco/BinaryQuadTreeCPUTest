# Interactive Playground

Interactive GUI applications for Conway's Game of Life and Dungeon Generator with full controls and visualization.

## Quick Start

```bash
# Conway's Game of Life
python Playground.py life

# Dungeon Generator
python Playground.py dungeon
```

## Conway's Game of Life Playground

### Features

- **50x50 Interactive Grid** - Draw and erase cells with mouse
- **Pattern Library** - 20+ classic patterns including:
  - Still lifes: Block, Beehive, Loaf, Boat, Tub
  - Oscillators: Blinker, Toad, Beacon, Pulsar, Pentadecathlon
  - Spaceships: Glider, LWSS, MWSS, HWSS
  - Methuselahs: R-pentomino, Diehard, Acorn
  - Guns: Gosper Glider Gun
- **Playback Controls** - Start, Pause, Stop, Step, Clear, Random
- **Speed Control** - 1-60 generations per second
- **Keyboard Shortcuts**:
  - `Space` - Start/Pause
  - `C` - Clear
  - `R` - Randomize
  - `S` - Step
  - `Q` - Quit

### Mouse Controls

| Action | Control |
|--------|---------|
| Draw cells | Left-click + drag |
| Erase cells | Right-click + drag |
| Zoom | Scroll wheel (future) |

### Usage Examples

```bash
# Start with default 50x50 grid
python Playground.py life

# Custom grid size (modify source)
# Edit: ConwayPlayground(grid_size=80, cell_size=8)
```

## Dungeon Generator Viewer

### Features

- **40x40 Grid** - Visual dungeon display
- **6 Generation Phases**:
  1. Initial Noise - Random seeding (45% walls)
  2. Cellular Automaton - Smoothing caves
  3. Room Expansion - Growing floor areas
  4. Corridor Carving - Connecting rooms
  5. Door Placement - Adding doorways
  6. Final Polish - Removing artifacts
- **Step-through Controls** - Advance one tick at a time
- **Auto-play Mode** - Watch generation automatically
- **Export** - Save dungeon as JSON for use in games
- **Visual Legend** - Color-coded tile types

### Tile Types

| Color | Type | Description |
|-------|------|-------------|
| ⬛ Black | Void | Empty/unused space |
| ⬜ White | Wall | Solid stone walls |
| 🟩 Green | Floor | Walkable room floor |
| 🟥 Red | Door | Doorway between rooms |
| 🟦 Blue | Corridor | Hallway connecting areas |

### Controls

| Button | Action |
|--------|--------|
| ▶ Auto | Toggle auto-play |
| ⟳ Step | Advance one tick |
| ⟲ Reset | Start over |
| ⏭ Finish | Jump to end |
| 💾 Export | Save as JSON |

### Export Format

```json
{
  "size": 40,
  "tick": 80,
  "grid": [[0,1,2,...], ...],
  "legend": {
    "0": "void",
    "1": "wall",
    "2": "floor",
    "3": "door",
    "4": "corridor"
  }
}
```

## Pattern Library (Conway's Life)

### Still Lifes
Patterns that don't change:
- **Block** - 2x2 square (4 cells)
- **Beehive** - Oval shape (6 cells)
- **Loaf** - Irregular oval (7 cells)
- **Boat** - L-shape with tail (5 cells)
- **Tub** - Diamond with hole (5 cells)

### Oscillators
Patterns that cycle through states:
- **Blinker** - Line of 3, period 2 (3 cells)
- **Toad** - Two rows offset, period 2 (6 cells)
- **Beacon** - Two 2x2 blocks, period 2 (8 cells)
- **Pulsar** - Large symmetric, period 3 (48 cells)
- **Pentadecathlon** - Long line, period 15 (10 cells)

### Spaceships
Patterns that move across the grid:
- **Glider** - Moves diagonally, period 4 (5 cells)
- **LWSS** - Light spaceship, period 4 (9 cells)
- **MWSS** - Medium spaceship, period 4 (12 cells)
- **HWSS** - Heavy spaceship, period 4 (15 cells)

### Methuselahs
Small patterns that evolve for many generations:
- **R-pentomino** - Evolves 1103 generations (5 cells)
- **Diehard** - Disappears after 130 generations (7 cells)
- **Acorn** - Grows large then stabilizes (7 cells)

### Guns
Patterns that create spaceships:
- **Gosper Glider Gun** - Produces gliders every 30 ticks (36 cells)

## Tips

### Conway's Life

1. **Start with patterns** - Insert a Glider or Gosper Glider Gun to see interesting behavior
2. **Try random** - Click "Random" for unpredictable evolution
3. **Watch collisions** - Insert two patterns close together
4. **Discover still lifes** - Some random patterns stabilize

### Dungeon Generator

1. **Watch the phases** - Each phase transforms the dungeon
2. **Step through slowly** - See how cellular automata creates caves
3. **Export for games** - Use JSON output in roguelike games
4. **Multiple runs** - Each generation is unique

## Dependencies

```bash
pip install matplotlib tkinter
```

## Troubleshooting

### Window doesn't appear
- Ensure tkinter is installed: `pip install tk` (Linux: `apt install python3-tk`)
- Check matplotlib backend: `matplotlib.use('TkAgg')`

### Patterns not inserting correctly
- Make sure you're clicking in the grid area
- Some large patterns (Pulsar, Gosper Gun) may be clipped at edges

### Export not working
- Check file permissions
- Ensure destination folder exists

## Integration with Games

### Using Exported Dungeons

```python
import json

with open('dungeon.json', 'r') as f:
    data = json.load(f)

grid = data['grid']
size = data['size']

# Convert to game format
for y in range(size):
    for x in range(size):
        tile = grid[y][x]
        if tile == 1:
            set_tile(x, y, 'wall')
        elif tile == 2:
            set_tile(x, y, 'floor')
        # etc.
```

### Using Conway Patterns

```python
# Pattern coordinates relative to insertion point
GLIDER = [(1,0), (2,1), (0,2), (1,2), (2,2)]

def insert_pattern(grid, pattern, cx, cy):
    for dx, dy in pattern:
        x, y = cx + dx, cy + dy
        if 0 <= x < len(grid[0]) and 0 <= y < len(grid):
            grid[y][x] = 1  # Alive
```
