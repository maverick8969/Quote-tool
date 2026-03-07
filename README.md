# Belt Order Layout Optimizer

A single-file, zero-dependency web application that helps manufacturers determine the most efficient belt widths and lengths to produce, minimizing material waste across multiple customer orders.

## What It Does

Given a list of customer belt orders (each with a width and length), the tool automatically groups them onto the fewest possible manufacturing runs using a **Best Fit Decreasing (BFD) bin-packing algorithm**. It displays a visual layout for each belt, calculates material waste, and flags potential issues.

## Features

- **Manual order entry** — Add orders one at a time via a form (Order ID, Width, Length)
- **Bulk import** — Paste CSV rows directly or upload a `.csv` file (with drag-and-drop support)
- **Optimizer** — Groups orders onto belts using Best Fit Decreasing on width, minimizing production runs
- **Visual belt layout** — Color-coded bar showing how each order occupies the belt width
- **Waste analysis** — Per-belt and overall waste in square feet and percentage
- **Infeasible order detection** — Orders exceeding the 108″ max width are flagged separately
- **Smart notes** — Contextual recommendations (e.g., high-waste belts, min-width constraints)

## Constraints & Business Rules

| Parameter | Value |
|---|---|
| Max belt width | 108 inches |
| Min belt width | 48 inches |
| Width margin added per order | +0.5 inches |
| Length margin added per order | +0.5 feet |
| Order width range | 6 – 108 inches |
| Order length range | 1 – 600 feet |

## How the Optimizer Works

1. Each order's **required width** = `order width + 0.5"` and **required length** = `order length + 0.5'`
2. Orders are sorted by required width (descending), then by required length (descending) — Best Fit Decreasing
3. Each order is placed into the existing bin (belt) where it fits with the least remaining slack; if no bin fits, a new belt is opened
4. The manufactured belt width is `max(48", sum of required widths on that belt)`
5. The manufactured belt length is the longest required length among all orders on that belt
6. Waste = `(belt width − sum of required widths) × belt length`

## File Structure

```
Quote-tool/
├── index.html   # Entire application — HTML, CSS, and JavaScript in one file
└── README.md
```

The entire application lives in `index.html` with no external dependencies, build steps, or server requirements.

## Usage

Open `index.html` directly in any modern browser:

```
file:///path/to/Quote-tool/index.html
```

Or serve locally:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

### CSV Format

Upload or paste orders in the following format (header row is auto-detected and skipped):

```
ID,Width,Length
ORD-001,24,300
ORD-002,48,200
,10,150
```

- **3 columns**: `ID, Width (in), Length (ft)`
- **2 columns**: `Width (in), Length (ft)` — ID is auto-generated
- Header row is optional and automatically skipped

## Code Overview

| Section | Location | Description |
|---|---|---|
| State | `orders[]`, `nextSeq` | In-memory order list and auto-increment counter |
| `addOrder()` | JS | Validates and adds a single order from the form |
| `handleCsvFile()` / `parseCsvText()` | JS | File upload and CSV parsing with header detection |
| `bulkAdd()` | JS | Parses pasted CSV text |
| `renderOrders()` | JS | Rebuilds the orders table in the DOM |
| `optimizeBelts()` | JS | Core BFD algorithm — returns `{ belts, infeasible }` |
| `runOptimize()` | JS | Calls the optimizer and triggers all render functions |
| `renderInfeasible()` | JS | Shows orders that exceed the 108″ max |
| `renderSummary()` | JS | Top-level stat cards (belt count, total material, waste) |
| `renderBelts()` | JS | Per-belt visual bar and order table |
| `renderNotes()` | JS | Contextual notes and recommendations |
