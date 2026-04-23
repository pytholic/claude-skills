---
name: excalidraw-diagrams
description: Draw polished, hand-drawn-style Excalidraw diagrams for any technical or conceptual topic. Use this skill whenever the user asks to "draw", "diagram", "visualise", "make an Excalidraw", or "sketch" something. Triggers include: system architecture, data flows, pipelines, concept maps, process flows, comparisons, and any request for a visual explanation. Always run the full three-phase approach: ASCII preview → wait for approval → draw. Never skip to drawing without the ASCII step first.
---

# Excalidraw Diagrams

## Core Principle

**SHOW BEFORE DRAW. ALWAYS PRODUCE AN ASCII SKETCH FIRST AND WAIT FOR EXPLICIT USER APPROVAL BEFORE DRAWING.**

"Looks good", "go ahead", "yes", "draw it" count as approval. Silence or a follow-up question does not.

---

## Phase 1: ASCII Preview

Sketch the diagram in ASCII. Choose layout direction first:

| Left → Right | Top → Down |
|---|---|
| Pipelines, data flows, layered systems (User / Agent / Storage) | Hierarchies, decision trees, sequential steps |
| 2–4 distinct layers | 4+ sequential steps or branching |

State the choice explicitly, e.g. *"Using left-to-right — three-layer system."*

**ASCII conventions:**
```
[Box]        node
[[ Zone ]]   background container / group
---->        arrow
- - ->       dashed arrow (secondary/indirect)
[i]          side annotation box
```

After the ASCII, state:
- Layout direction and why
- Color zones planned (one per logical group)
- Any side annotation boxes

Close with: *"Does this look right? Any changes before I draw?"*

**Do not proceed until the user explicitly approves.** If they request changes, revise the ASCII and ask again.

---

## Phase 2: Draw

Call `Excalidraw:read_me` first to load the element format, then call `Excalidraw:create_view`.

**Style preferences to apply:**

*Fonts:* Always `fontFamily: 1` (Excalifont) for all text — titles, zone labels, node labels, annotations. Use `fontFamily: 3` (monospace) only for command/code names like `kb:compile`.

*Font sizes:* titles ≥ 20, node labels ≥ 16, annotations ≥ 14. Never below 14.

*Roughness:* Always `roughness: 1`. Never 0 (sterile) or 2 (messy).

*Zone backgrounds:* One per logical group. Always `opacity: 35`, `fillStyle: "solid"`, `roundness: { type: 3 }`.

| Zone | Background | Stroke |
|------|-----------|--------|
| Input / User | `#dbe4ff` | `#4a9eed` |
| Agent / Logic | `#e5dbff` | `#8b5cf6` |
| Storage / Knowledge | `#d3f9d8` | `#22c55e` |
| Output / Result | `#c3fae8` | `#06b6d4` |
| Annotation / Note | `#fff3bf` | `#f59e0b` |

*Node fills* (inside zones):

| Node type | Fill |
|-----------|------|
| Input | `#a5d8ff` |
| Agent command | `#d0bfff` |
| Storage | `#b2f2bb` |
| Output | `#c3fae8` |
| Note/callout | `#fff3bf` |
| Neutral | `#e9ecef` |

*Arrows:* `strokeWidth: 2` for primary, `strokeWidth: 1` + `strokeStyle: "dashed"` for secondary. Match stroke color to source node's zone color.

*Side annotation boxes:* Place outside the main flow (right or below) with a dashed leader arrow in `#f59e0b`. Use for "more info" that would clutter the main diagram.

**Drawing order:**
1. `cameraUpdate` (always first)
2. Title + subtitle
3. Zone backgrounds (back to front)
4. Zone labels (top-left inside each zone)
5. Nodes (left→right, top→bottom within zone)
6. Arrows
7. Side annotation boxes
8. Final `cameraUpdate` zoom-out

Use intermediate `cameraUpdate` calls to pan into each zone as you draw it — this makes the drawing feel alive as it streams in.

---

## Process Violations

- Drawing before receiving explicit approval
- Treating silence as approval
- Re-drawing after ASCII revision without asking again
- More than 4 distinct node fill colors (visual noise)
- Font sizes below 14
- Camera sizes not exactly S/M/L/XL/XXL (non-4:3 distorts)
- Separate text elements where a labeled shape would worki
