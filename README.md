# BinaryQuadTreeCPUTest

**Two experiments in the same idea: space as a recursive language.**

> A fractal grammar engine where 4-bit masks program geometry â and a binary image codec that uses the same spatial logic to beat JPEG compression.

![GEO grammar engine â spiral.geo running at depth 6](media/geo_spiral.gif)

![GEOI vs JPEG compression comparison](media/comparison.png)

---

## What Is This?

This repo contains two separate but deeply related systems built on a single insight:
**a square can always be divided into four smaller squares, and that fact is a complete computational primitive.**

### 1. The GEO Grammar Engine *(Python â proof of concept)*

A declarative scripting language where you program **how fractal geometry evolves over time**.
Each node in a recursive quadtree carries a 4-bit mask. The mask controls which quadrants are
drawn and subdivided. Rules fire every tick to change the mask â switching loop families,
reacting to depth, time, neighbors, probability. The result is living geometry that rotates,
pulses, spreads, and self-organizes.

```geo
NAME   spiral
RULE   IF tick%8=0   THEN SWITCH Y_LOOP    AS beat-Y
RULE   IF tick%8=2   THEN SWITCH X_LOOP    AS beat-X
RULE   IF tick%8=4   THEN SWITCH Z_LOOP    AS beat-Z
RULE   IF tick%8=6   THEN SWITCH DIAG_LOOP AS beat-D
RULE   IF depth>=5   THEN GATE_ON          AS seal-deep
DEFAULT ADVANCE
```

### 2. The `.geoi` / `.geov` Compression Codec *(Go â the real product)*

A binary image and video compression format that uses the **same quadtree spatial subdivision**
as the grammar engine â but for image compression. Large uniform regions collapse to single
nodes. Only areas with actual detail get subdivided. The result: **better compression than JPEG
on images with large uniform regions** (skies, walls, illustration, pixel art).

```
Raw 512Ã512:      1,048,576 bytes
JPEG q=95:           63,482 bytes  (16.5x)
.geoi q=248:         44,100 bytes  (23.8x) â beats JPEG at this quality level
.geoi q=245:         20,200 bytes  (52.0x) â matches JPEG quality at 4x smaller
```

---

## The Core Idea

Every quadrant knows its address. The address *is* the data structure.

```
Z-order (Morton) curve â maps 2D position to 1D index:

  y=1:  [ 2  3  6  7 ]
  y=0:  [ 0  1  4  5 ]
          x=0 x=1 x=2 x=3

Bit-interleave (x=2, y=1):  x=10b, y=01b â Morton code 0110b = 6
```

This spatial locality means:
- **Progressive decode**: stop reading the bitstream at any depth â valid lower-resolution image
- **Adaptive detail**: each region gets exactly as many bits as it needs
- **No block artifacts**: no 8Ã8 DCT blocks â regions are as large or small as the image demands

---

## Quick Start (Grammar Engine)

```bash
git clone https://github.com/sfdimarco/BinaryQuadTreeCPUTest.git
cd BinaryQuadTreeCPUTest
pip install -r requirements.txt

python BinaryQuadTreeTest.py                              # self-organising grid
python BinaryQuadTreeTest.py --geo examples/spiral.geo   # load a .geo script
python BinaryQuadTreeTest.py --list                       # see all built-in demos
```

## Quick Start (Go Codec)

```bash
cd go
go build ./cmd/geocoder

# Encode an image
./geocoder encode -i photo.png -o photo.geoi -q 245

# Decode (full resolution)
./geocoder decode -i photo.geoi -o photo_out.png

# Progressive decode at half resolution
./geocoder decode -i photo.geoi -o photo_thumb.png -d 7

# Full benchmark vs JPEG
./geocoder bench -i photo.png -q 245

# File info
./geocoder info -i photo.geoi
```

Run all tests:
```bash
cd go
go test ./...
```

---

## Architecture

### The Grammar Engine (Python)

Three stacked layers:

**Layer 1 â Mask Engine**: 16 possible 4-bit mask values, partitioned into five loop families.
Each family is a closed cycle. `ADVANCE` steps forward one position.

| Family | Cycle | Feel |
|--------|-------|------|
| **Y_LOOP** | `1000â0100â0010â0001` | Single quadrant orbits |
| **X_LOOP** | `1100â0101â0011â1010` | Adjacent pair cycles |
| **Z_LOOP** | `0111â1011â1101â1110` | Three-quadrant sweep |
| **DIAG_LOOP** | `1001â0110` | Diagonal pair toggles |
| **GATE** | `0000` / `1111` | Fixed / frozen |

**Layer 2 â Grammar Programs**: Ordered `IF condition THEN action` rules. First match wins.
Conditions compose with `AND`, `OR`, `BUT`, `NOT`. Turing-complete â branch on state, time,
depth, neighbor context, probability, cell variables.

**Layer 3 â Grid / CA**: An NÃM grid of quadtree roots, each with its own program. Cells read
neighbors, emit signals, vote on programs. Same-tick snapshot semantics prevent order artifacts.

### The Codec (Go)

```
PNG/JPEG input
     â
[BuildFromImage]  Load pixels, pad to power-of-2 square
     â
[buildRecursive]  Bottom-up quadtree construction
     â  Each region averages its 4 children's YCbCr colors
     â  canPrune(): if all 4 children are leaves AND colors within quality threshold â merge
     â  computeDelta(): child color = parent average + small delta
     â
[QuadNode tree]   Leaf nodes = uniform regions. Internal = subdivided.
     â
[EncodeHuffman]   Pass 1: collect delta distribution. Build per-channel Huffman tables.
     â             Pass 2: write header + root color + 4 Huffman tables + coded bitstream
     â
[.geoi file]      ~3-50x smaller than raw pixels, competitive with JPEG
```

**Why YCbCr?** Separates luminance (Y) from chrominance (Cb, Cr). Human eyes are 4Ã less
sensitive to color than brightness â chroma channels get 2Ã the pruning threshold. Same trick
JPEG uses, applied to spatial quadtree deltas instead of DCT coefficients.

**Why delta encoding?** Child nodes store the difference from their parent's average, not
absolute colors. Deltas cluster near zero. Huffman codes frequent small deltas with 1-2 bits,
rare large deltas with longer codes. On typical images: v2 Huffman is 3Ã smaller than v1 raw.

**Progressive decode**: `Decode(reader, maxDepth=4)` stops at depth 4 â a valid 1/16-resolution
image. Same file. Same decoder. Just stop reading earlier.

---

## Go Codec: File Structure

```
go/
âââ go.mod                          # module github.com/sfdimarco/geo
âââ cmd/geocoder/main.go            # CLI: encode / decode / info / bench
âââ pkg/
    âââ morton/
    â   âââ morton.go               # Z-order curve encode/decode, child addressing
    â   âââ morton_test.go          # 8 tests + benchmarks (~3ns/op)
    âââ quadtree/
    â   âââ node.go                 # QuadNode, Color/YCbCr, ColorDelta
    â   âââ builder.go              # BuildFromImage, adaptive pruning, RenderToPixels
    â   âââ node_test.go            # uniform collapse, checkerboard, quadrant colors
    âââ codec/
        âââ huffman.go              # HuffmanTable, BitWriter, BitReader, tree serialization
        âââ format.go               # .geoi header, v1 raw encoder, v2 Huffman encoder/decoder
        âââ codec_test.go           # 10 tests: roundtrip v1+v2, progressive, bench
```

### File Format (v2 / Huffman)

```
ââââââââââââââââââââââââââââââââââââââââââââââââââââ
â HEADER (16 bytes)                                â
â   Magic[4] = 'GEOi'  Version=2  MaxDepth         â
â   ColorMode  Quality  Width(4)  Height(4)         â
ââââââââââââââââââââââââââââââââââââââââââââââââââââ¤
â ROOT COLOR (4 bytes: Y Cb Cr A)                  â
ââââââââââââââââââââââââââââââââââââââââââââââââââââ¤
â NODE COUNT (4 bytes)                             â
ââââââââââââââââââââââââââââââââââââââââââââââââââââ¤
â HUFFMAN TABLES Ã 4                               â
â   [Y deltas]  [Cb deltas]  [Cr deltas]  [masks]  â
â   Each: count(2) + entries Ã (sym+len+code)(6)   â
ââââââââââââââââââââââââââââââââââââââââââââââââââââ¤
â BITSTREAM LENGTH (4 bytes)                       â
ââââââââââââââââââââââââââââââââââââââââââââââââââââ¤
â HUFFMAN-CODED BITSTREAM                          â
â   For each node in Z-order depth-first:          â
â   [DY bits] [DCb bits] [DCr bits] [mask bits]    â
ââââââââââââââââââââââââââââââââââââââââââââââââââââ
```

---

## The `.geo` Language

`.geo` is a declarative grammar language for writing quadtree animation programs.
Full reference: [GEO_LANGUAGE.md](GEO_LANGUAGE.md)

```geo
NAME   heat_spread
DEFINE hot  var.heat >= 10
DEFINE warm var.heat >= 5
RULE   IF hot  THEN SWITCH Z_LOOP AND EMIT spread   AS boiling
RULE   IF warm THEN SWITCH X_LOOP AND INC_VAR heat  AS heating
RULE   IF signal(spread) THEN INC_VAR heat          AS absorb
DEFAULT ADVANCE
```

**35+ example scripts** in [`examples/`](examples/) â terrain generation, cellular automata,
cosmos simulations, animation cycles, self-organization patterns, and more.

---

## Examples

```bash
# Grammar engine examples
python BinaryQuadTreeTest.py --geo examples/spiral.geo
python BinaryQuadTreeTest.py --geo examples/terrain/caves.geo --grid
python BinaryQuadTreeTest.py --geo examples/selforg/voronoi.geo --grid
python BinaryQuadTreeTest.py --geo examples/cosmos_sim.geo

# Codec examples
cd go
./geocoder bench -i my_photo.png -q 245        # full comparison table vs JPEG
./geocoder encode -i art.png -o art.geoi -q 255  # lossless
./geocoder decode -i art.geoi -o art_out.png -d 6  # half-res progressive
```

---

## Progressive Decode

One `.geoi` file, four resolutions â stop reading the bitstream at any depth:

![Progressive decode â depth 5 through 9](media/progressive.png)

---

## Status

| Component | Status | Notes |
|-----------|--------|-------|
| GEO grammar engine | â Complete | Python, single file, 35+ example scripts |
| GEO language spec | â Complete | Full reference in GEO_LANGUAGE.md |
| GeoStudio IDE | â Complete | Built-in IDE with live preview |
| Morton/Z-order codec | â Complete | Go, ~3ns/op, fully tested |
| Quadtree builder (YCbCr) | â Complete | Adaptive pruning, delta encoding |
| Codec v1 (raw) | â Complete | Fixed 5 bytes/node, baseline |
| Codec v2 (Huffman) | â Complete | Per-channel Huffman, 2-4Ã over v1 |
| CLI geocoder tool | â Complete | encode/decode/info/bench commands |
| Progressive decode | â Complete | Stop at any depth â valid image |
| PSNR/SSIM quality metrics | ð Phase 4 | Perceptual quality vs JPEG |
| Streaming HTTP decoder | ð Phase 3 | Range requests â progressive web |
| Video codec (.geov) | ð Future | Inter-frame delta on quadtrees |

---

## License

[MIT](LICENSE)

---

## The STCP Hypothesis

> **GEO compression and Barnes-Hut N-body gravity are the same algorithm.**

The **Spatial Tolerance Compression Principle (STCP)** is the observation that these two systems â a fractal image codec and a galaxy simulator â share an identical computational structure:

| GEO Image Codec | Barnes-Hut N-body |
|---|---|
| Subdivide until region is uniform | Subdivide until node is "far enough" |
| `pruning threshold` (quality) | `Î¸` parameter (0.0â1.0) |
| Leaf node = single color | Leaf node = center of mass |
| Stop recursing = compress | Stop recursing = approximate force |

Both systems ask the same question at every quadtree node: *"Is this region uniform enough to treat as a single thing?"* The answer â and the threshold â is structurally identical.

### QJL â Second-Order STCP

**QJL (Quantized Joint Leverage)** extends Barnes-Hut by quantizing the spherical coordinates (Î¸, Ï) of accepted force interactions, enabling force vector **caching across frames**. Particles in similar positions relative to a cluster reuse previously computed force angles rather than recomputing them.

Validated result: **~45% cache hit rate** on 8,000-particle N-body simulations.

### Live Simulations

Interactive demos archived in [`research-/simulations`](https://github.com/sfdimarco/research-/tree/master/simulations):

- **[Universe1-Basic](https://github.com/sfdimarco/research-/blob/master/simulations/Universe1-Basic.html)** â Barnes-Hut N-body baseline (STCP without QJL)
- **[Universe2-Benchmark](https://github.com/sfdimarco/research-/blob/master/simulations/Universe2-Benchmark.html)** â QJL vs exact force benchmark with live timing HUD and KE drift tracking
- **[Universe3-Cache](https://github.com/sfdimarco/research-/blob/master/simulations/Universe3-Cache.html)** â Cache hit rate visualizer; validates the ~45% figure live

> Download and open any `.html` file â no server needed. Press **Q** to toggle QJL on/off, **R** to reset stats.
