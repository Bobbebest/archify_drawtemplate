---
name: archify
description: Generate beautiful, professional architecture and technical diagrams as standalone HTML files with dark/light theme toggle and PNG/JPEG/WebP/SVG export. Use this skill whenever the user asks for: system architecture diagrams, infrastructure diagrams, cloud architecture (AWS/GCP/Azure), microservices diagrams, technical workflow diagrams, approval flows, CI/CD pipelines, runbooks, incident response flows, API call sequences, data pipeline diagrams, ETL/ELT maps, state machine diagrams, lifecycle diagrams, or asks to "draw", "visualize", "diagram", "chart", "map out" any technical system or process. Also use when user wants to convert or beautify a Mermaid diagram. Always prefer this over plain SVG or Mermaid when the user wants a polished, exportable diagram.
license: MIT
metadata:
  version: "5.0"
  author: tt-a1i
  based_on: Cocoon-AI/architecture-diagram-generator (MIT, v1.0)
  changelog: "v5.0 — Added QG8 breathing room & arrow visibility (min gap 60px, recommended 80px)"
---

# Archify Skill

Create professional technical diagrams as **self-contained HTML files** with inline SVG, a dark/light theme toggle, and a built-in export menu (PNG / JPEG / WebP / SVG at up to 4× resolution).

## Quick Start

1. Read the description → pick diagram type (see table below)
2. Build the SVG directly into `assets/template.html` following the Design System
3. Run the mandatory Self-Review Checklist — **all 7 Quality Gates must pass**
4. Deliver the finished HTML file

> **No Node.js renderers available in this environment.** Always use the hand-placed SVG approach with `assets/template.html` as base.

---

## Diagram Type Selection

| Type | Use for | Key signal words |
|------|---------|-----------------|
| `architecture` | System components, cloud resources, services, security boundaries | "architecture", "system", "infrastructure", "cloud", "microservices" |
| `workflow` | Technical flows, approval gates, CI/CD, runbooks, incident response | "workflow", "flow", "process", "approval", "CI/CD", "pipeline" |
| `sequence` | API call chains, request lifecycles, cache fallback, async traces | "sequence", "call chain", "who calls whom", "request lifecycle" |
| `dataflow` | Data pipelines, ETL/ELT, PII isolation, data lineage | "data flow", "pipeline", "ETL", "lineage", "PII", "data map" |
| `lifecycle` | State machines, status transitions, retries, terminal states | "state", "lifecycle", "status", "state machine", "retry" |

---

## ⚠️ FOUR QUALITY GATES — Must pass before delivering

These fix the four most common diagram failures. Check all four before output.

---

### Quality Gate 1 — Arrow Connection Accuracy

**Rule: Every arrow must originate from and terminate at the exact edge midpoint of its source and target box.**

SVG arrows have no awareness of box geometry. You must calculate edge coordinates manually using box `x`, `y`, `width`, `height`:

```
Box edge midpoints (given rect at x, y, width w, height h):
  left-center:   (x,         y + h/2)
  right-center:  (x + w,     y + h/2)
  top-center:    (x + w/2,   y)
  bottom-center: (x + w/2,   y + h)
```

**Workflow: For every edge you draw:**
1. Identify exit side on source box → compute (x1, y1)
2. Identify enter side on target box → compute (x2, y2)
3. Write those computed values into `x1=` `y1=` `x2=` `y2=`

**NEVER estimate arrow coordinates by eye.** If a box is at x=120, width=110, the right edge is exactly x=230, not approximately 225 or 240.

For `<line>` arrows ending with `marker-end`, shorten x2/y2 by 8px in the arrow direction so the line ends just before the arrowhead polygon:

```svg
<!-- Box B: x=300 y=100 w=110 h=50  → left edge = x=300, y=125 -->
<!-- Box A: x=120 y=100 w=110 h=50  → right edge = x=230, y=125 -->
<!-- Arrow A→B, horizontal: shorten x2 by 8 (arrowhead offset) -->
<line x1="230" y1="125" x2="292" y2="125"
      class="a-emphasis" stroke-width="1.5"
      marker-end="url(#arrowhead-emphasis)"/>
```

For diagonal or multi-bend paths, use `<path>` with computed waypoints and shorten the last segment endpoint by 8px.

---

### Quality Gate 2 — Uniform Node Size Per Row

**Rule: All nodes in the same swimlane row or architectural tier must share identical `width` and `height`.**

Nodes that look different sizes in the same row break visual rhythm and signal careless layout. The rule:

- Before placing nodes, decide ONE standard box size per lane/tier: e.g., `width=110 height=50`
- Apply that size to every node in that lane
- Only deviate if a node has significantly more text lines (then use a larger STANDARD size for that lane, applied uniformly)
- START/END terminal nodes may use a rounded-pill shape (`rx=height/2`) but must still use a consistent width

**Sizing tiers (pick one per lane, apply uniformly):**

| Lane complexity | Recommended size |
|----------------|-----------------|
| Single-line label | `width=100 height=36` |
| Two-line label (name + subtitle) | `width=110 height=50` |
| Three-line label | `width=120 height=62` |

**Example — correct uniform sizing in a swimlane:**
```svg
<!-- All nodes in Lane 1 use width=110 height=50 -->
<rect x="90"  y="115" width="110" height="50" rx="6" class="c-mask"/>
<rect x="90"  y="115" width="110" height="50" rx="6" class="c-backend" stroke-width="1.5"/>

<rect x="250" y="115" width="110" height="50" rx="6" class="c-mask"/>
<rect x="250" y="115" width="110" height="50" rx="6" class="c-backend" stroke-width="1.5"/>

<rect x="410" y="115" width="110" height="50" rx="6" class="c-mask"/>
<rect x="410" y="115" width="110" height="50" rx="6" class="c-backend" stroke-width="1.5"/>
```

---

### Quality Gate 3 — Arrow Layering (No Arrows Over Nodes)

**Rule: All arrows/paths MUST appear in SVG document order BEFORE all node `<rect>` and `<text>` groups.**

SVG paints elements in document order — later elements render on top. Arrows written after nodes will overdraw box fills and text, making the diagram unreadable.

**Mandatory document order:**
```
1. <defs> (markers, patterns)
2. Background grid rect
3. Swimlane / boundary rects  ← lanes drawn first
4. ALL <line> and <path> arrows  ← arrows before everything
5. ALL node groups (c-mask rect + c-type rect + text)  ← nodes on top
6. Lane label texts (rotated)
7. Phase header bars
8. Legend block
```

**NEVER** place a `<line>` or `<path>` arrow after a node group — even for "just one connection added later". If you need to add an arrow, insert it into the arrows section, not at the end of the SVG.

The two-rect (`c-mask` + `c-type`) pattern ensures the opaque mask hides any arrow that passes behind the node, so the arrow appears to cleanly terminate at the box edge.

---

### Quality Gate 4 — Legend Separation

**Rule: The legend block must be visually separated from the diagram content with ≥40px clearance and a distinct background.**

A legend merged into the diagram content area confuses readers. Requirements:

1. **Vertical gap**: legend top ≥ (lowest diagram element bottom + 40px)
2. **Background panel**: legend must have its own `c-mask` + `c-external` rect background (not just floating text)
3. **Internal padding**: legend content starts ≥12px inside the panel edges
4. **Legend height**: panel height ≥ 56px (enough for items + footnote)
5. **viewBox extension**: extend viewBox height to accommodate legend + 20px bottom margin

**Legend template:**
```svg
<!-- Legend panel — placed AFTER all diagram content -->
<!-- legendY = (lowest element bottom) + 40 -->
<rect x="80" y="[legendY]"    width="[W-100]" height="60" rx="6" class="c-mask"/>
<rect x="80" y="[legendY]"    width="[W-100]" height="60" rx="6" class="c-external" stroke-width="1"/>

<!-- Color swatches row at legendY+14 -->
<rect x="100" y="[legendY+14]" width="14" height="10" rx="2" class="c-mask"/>
<rect x="100" y="[legendY+14]" width="14" height="10" rx="2" class="c-backend" stroke-width="1"/>
<text x="120" y="[legendY+23]" class="t-muted" font-size="8">Team / PIC</text>

<!-- ... more swatches at +130px intervals ... -->

<!-- Arrow style samples at legendY+14, after color swatches -->
<line x1="[lx]"    y1="[legendY+19]" x2="[lx+22]" y2="[legendY+19]"
      class="a-emphasis" stroke-width="1.5" marker-end="url(#arrowhead-emphasis)"/>
<text x="[lx+26]" y="[legendY+23]" class="t-muted" font-size="8">Primary flow</text>

<!-- Footnote at legendY+44 -->
<text x="100" y="[legendY+44]" class="t-dim" font-size="7">
  HĐ = Hợp đồng  ·  BBNT = Biên bản nghiệm thu  ·  ĐNTT = Đề nghị thanh toán
</text>
```

---

### Quality Gate 5 — No Node Overlap

**Rule: No two nodes may overlap each other. Every node must have clear whitespace separation on all sides.**

Overlapping nodes are the most common layout defect from manually placed SVG coordinates. They occur when spacing is not calculated before placement.

**Before placing any node, compute its bounding box and verify it does not intersect any existing node:**

```
Node bounding box:  left = cx - w/2,  right = cx + w/2,  top = cy - h/2,  bottom = cy + h/2
Minimum clearance:  ≥16px horizontal gap,  ≥12px vertical gap between any two nodes
```

**Spacing calculation workflow — do this before writing any node SVG:**

1. Fix a standard node size per lane (e.g., w=110, h=50)
2. Fix a horizontal step = node_w + gap (e.g., 110 + 20 = 130px between centers)
3. Fix a vertical step = node_h + gap (e.g., 50 + 30 = 80px between rows)
4. List all cx values for a lane: cx₁, cx₁+130, cx₁+260 ...
5. Verify: cx_n - cx_{n-1} ≥ node_w + 16 for every adjacent pair

**Common mistake to avoid:**

```svg
<!-- WRONG ❌ — nodes overlap: second node starts at 200, first ends at 210 -->
<rect x="100" y="100" width="110" height="50"/>  <!-- right edge = 210 -->
<rect x="200" y="100" width="110" height="50"/>  <!-- left edge = 200 — OVERLAP -->

<!-- CORRECT ✅ — gap of 20px between nodes -->
<rect x="100" y="100" width="110" height="50"/>  <!-- right edge = 210 -->
<rect x="230" y="100" width="110" height="50"/>  <!-- left edge = 230 — 20px clear -->
```

**For swimlane layouts, nodes in different lanes at the same column must also not overlap vertically:** lane top + padding ≥ previous lane bottom.

---

### Quality Gate 6 — Arrow Completeness & Shape Semantics

#### 6A — Every Arrow Must Connect Two Nodes

**Rule: Every `<line>` or `<path>` arrow must have both endpoints connected to a node. Dangling arrows (one end floating in empty space) are forbidden.**

Before writing each arrow, verify:
- `(x1, y1)` lies exactly on an edge midpoint of a **source node** (computed from that node's coordinates)
- `(x2, y2)` lies exactly on an edge midpoint of a **target node** (computed, minus 8px arrowhead offset)
- Both source and target nodes are defined in this diagram — no arrows pointing to empty space

**Arrow completeness checklist (run per arrow):**
```
Source node ID:   _____________   exit side: top / bottom / left / right
Target node ID:   _____________   enter side: top / bottom / left / right
x1 computed as:   source.cx ± source.w/2   OR   source.cx, source.cy ± source.h/2
x2 computed as:   target.cx ± target.w/2   OR   target.cx, target.cy ± target.h/2  − 8px offset
```

If either node ID is blank → **do not draw the arrow**.

#### 6B — Use Correct Flowchart Shape Semantics

**Rule: Use the standard flowchart shape for each node type. Never use a plain rectangle for start/end or decision nodes.**

| Node type | Shape | SVG implementation |
|-----------|-------|--------------------|
| **Start / End** | Oval / pill (rounded ends) | `<rect rx="height/2">` — fully rounded |
| **Process / Step** | Rectangle | `<rect rx="6">` — lightly rounded corners |
| **Decision** | Diamond | `<polygon points="cx,top  right,cy  cx,bottom  left,cy">` |
| **Input / Output** | Parallelogram | `<polygon>` with skewed x offsets |
| **Document / File** | Rectangle + wavy bottom | `<path>` with sine-curve bottom edge |

Minimum rule: **Start and End nodes must always use pill shape** (`rx = height/2`). Decision nodes that branch the flow must use diamond shape.

```svg
<!-- START node: pill shape -->
<rect x="[cx-50]" y="[cy-18]" width="100" height="36" rx="18" class="c-mask"/>
<rect x="[cx-50]" y="[cy-18]" width="100" height="36" rx="18" class="c-backend" stroke-width="1.5"/>

<!-- DECISION node: diamond -->
<rect x="[cx-40]" y="[cy-40]" width="80" height="80" rx="4" class="c-mask"/>  <!-- mask as bounding box -->
<polygon points="[cx],[cy-38]  [cx+38],[cy]  [cx],[cy+38]  [cx-38],[cy]"
         class="c-security" stroke-width="1.5"/>
```

---

### Quality Gate 7 — Alignment & Equal Spacing

**Rule: Nodes in the same row must share the same center-Y. Nodes in the same column must share the same center-X. Spacing between adjacent nodes must be equal within a lane.**

From flowchart best practice: *misaligned or unequally spaced symbols make a diagram look messy and hard to follow.*

**Requirements:**

1. **Row alignment**: all nodes in the same horizontal flow → identical `cy` value. Never offset one node 3px up or down from its neighbors.

2. **Column alignment**: nodes that are vertically stacked across lanes → identical `cx` value. This makes vertical arrows perfectly straight (no diagonal drift).

3. **Equal spacing**: the gap between node A right-edge and node B left-edge must equal the gap between node B right-edge and node C left-edge within the same lane row.
   ```
   gap_AB = (cx_B - w/2) - (cx_A + w/2)
   gap_BC = (cx_C - w/2) - (cx_B + w/2)
   gap_AB must equal gap_BC  (±2px tolerance)
   ```

4. **Minimum spacing**: horizontal gap ≥ 16px; vertical gap between lanes ≥ 20px.

5. **Flow direction**: primary flow reads **left → right** on the same row, then **top → bottom** across rows. Avoid rightward flows that suddenly go left unless explicitly a feedback/retry loop.

**Pre-layout grid approach (recommended):**
```
Define column centers:  colX[] = [LABEL_W + node_w/2 + col_i * (node_w + gap)]
Define row centers:     rowY[] = [lane_top + lane_h/2]  (one row per lane, or explicit if multi-row)
Place every node at (colX[i], rowY[j]) — never deviate without a documented reason
```

---

### Quality Gate 8 — Breathing Room & Arrow Visibility

**Rule: Node spacing must be wide enough that arrows are clearly visible as distinct connectors — not just thin slivers between boxes.**

The 16px minimum from QG5 is a collision floor, not a design target. Arrows that are too short look like errors or rendering artifacts.

**Minimum spacing ratios:**

| Context | Min gap between node edges | Recommended |
|---------|---------------------------|-------------|
| Horizontal (same row) | 48px | **60–80px** |
| Vertical (same row, different rows) | 40px | **60–80px** |
| Cross-lane vertical connectors | 30px per boundary | 40px+ |

**Column step formula (use this, not QG5's 16px floor):**
```
col_step = node_width + h_gap

h_gap = 60px minimum, 80px recommended
col_step = 120 + 80 = 200px  ← use this as default for w=120 nodes

Example column centers (LABEL_W=100, first node centered at 160):
  col0=160, col1=360, col2=560, col3=760, col4=960, col5=1160 ...
  edge gap = 200 - 120 = 80px ✓ — arrows will have ~72px visible length
```

**Row step formula:**
```
row_step = node_height + v_gap
v_gap = 60px minimum

Example: h=52, v_gap=60 → row_step=112
  row1_cy = lane_top + top_pad + h/2  (top_pad ≥ 24px)
  row2_cy = row1_cy + 112
```

**Lane height formula:**
```
single-row lane:  height = node_h + 2×padding  (padding ≥ 28px each side)  → h + 56 minimum
two-row lane:     height = 2×node_h + v_gap + 2×padding                    → 2h + gap + 56
```

**Arrow minimum visible length — hard check before finalising:**
```
horizontal arrow visible length = gap - 8  (arrowhead offset)
  must be ≥ 40px; if not → increase col_step

vertical arrow visible length = gap - 8
  must be ≥ 40px; if not → increase row_step or lane height
```

**Bad vs good example:**
```
BAD  (v4 style): col_step=144, node_w=120 → gap=24px → arrow only 16px visible ❌
GOOD (v5 style): col_step=200, node_w=120 → gap=80px → arrow  72px visible ✓
```

 **Hardcoded `fill="#22d3ee"` will NOT update on theme change.**

```svg
<!-- WRONG ❌ -->
<rect fill="rgba(6,78,59,0.4)" stroke="#34d399"/>

<!-- CORRECT ✅ -->
<rect class="c-mask"/>
<rect class="c-backend" stroke-width="1.5"/>
<text class="t-primary" font-size="11" font-weight="600">API Server</text>
<text class="t-muted" font-size="9">FastAPI :8000</text>
```

---

## Design System

### Component Fill Classes
| Class | Color | Use for |
|-------|-------|---------|
| `c-frontend` | Cyan | Clients, UI, browsers, mobile |
| `c-backend` | Emerald | Services, APIs, workers |
| `c-database` | Purple | Databases, caches, stores |
| `c-cloud` | Amber | Managed cloud services, infrastructure |
| `c-security` | Rose | Auth, secrets, firewalls, encryption |
| `c-messagebus` | Orange | Kafka, RabbitMQ, SNS, queues |
| `c-external` | Slate | 3rd parties, generic external |
| `c-mask` | Opaque bg | Always placed BEFORE the styled rect |

### Text Classes
- `t-primary` — main labels
- `t-muted` — sublabels, secondary info
- `t-dim` — annotations, tiny hints
- `t-frontend`, `t-backend`, `t-database`, `t-cloud`, `t-security`, `t-messagebus`, `t-external` — colored accent text

### Arrow / Edge Classes
| Class | Style | Use for |
|-------|-------|---------|
| `a-default` | Solid gray | Standard connections |
| `a-emphasis` | Solid green | Hot paths, primary flows |
| `a-security` | Dashed rose | Auth flows, PII, policy |
| `a-dashed` | Dashed purple | Async, batch, optional |

Always pair with `marker-end="url(#arrowhead)"` or `url(#arrowhead-emphasis)` etc.

### Boundary Classes
- `c-region` — dashed amber box
- `c-security-group` — dashed rose box
- `c-lane` — swimlane background

### Typography
- JetBrains Mono (falls back to system monospace)
- Component names: `font-size="11"` with `font-weight="600"`
- Sublabels: `font-size="9"` normal weight
- Annotations / legend: `font-size="8"`
- Footnotes: `font-size="7"`

---

## Hard Layout Rules

1. **Two-rect pattern everywhere**: `c-mask` FIRST (identical geometry), then `c-<type>` on top.

2. **Document order**: defs → grid → lanes → **arrows** → **nodes** → labels → legend. No exceptions.

3. **Vertical stacking gap**: ≥40px between node rows. Message buses (20px tall) sit in gaps.

4. **Boundary padding**: boundary `y` = inner component `y` − 30; boundary `height` = inner `height` + 50.

5. **Legend clearance**: legend top = (max bottom of all nodes) + 40px minimum.

6. **viewBox sizing**: viewBox height = legend bottom + 20px. viewBox width = rightmost element right + 20px.

7. **Uniform sizing per row** (QG2): identical `width`/`height` for all nodes in the same lane.

8. **Computed arrow endpoints** (QG1): every `x1 y1 x2 y2` derived from box coordinates, never guessed.

9. **Breathing room** (QG8): horizontal gap between node edges ≥ 60px (recommended 80px); vertical gap ≥ 60px between rows. col_step = node_w + 80 minimum. Arrow visible length must be ≥ 40px after arrowhead offset.

10. **Arrow completeness** (QG6A): every arrow connects two named nodes. No dangling endpoints.

11. **Shape semantics** (QG6B): START/END = pill (`rx=h/2`), DECISION = diamond (`<polygon>`), PROCESS = rectangle.

12. **Grid alignment** (QG7): all nodes in same row → identical `cy`; same column → identical `cx`; equal spacing within a lane.

---

## Workflow / Swimlane Full Pattern

```svg
<svg viewBox="0 0 1100 [legendBottom+20]" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" class="m-default"/>
    </marker>
    <marker id="arrowhead-emphasis" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" class="m-emphasis"/>
    </marker>
    <marker id="arrowhead-dashed" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" class="m-dashed"/>
    </marker>
    <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
      <path d="M 40 0 L 0 0 0 40" class="c-grid" stroke-width="0.5"/>
    </pattern>
  </defs>

  <!-- 1. Background -->
  <rect width="100%" height="100%" fill="url(#grid)"/>

  <!-- 2. Swimlane backgrounds -->
  <rect x="80" y="50"  width="1000" height="100" rx="0" class="c-lane" stroke-width="1"/>
  <rect x="80" y="150" width="1000" height="100" rx="0" class="c-lane" stroke-width="1"/>

  <!-- 3. ALL ARROWS — computed from box coordinates -->
  <!--
    NodeA: x=100 y=65  w=110 h=50  → right-center=(210, 90)
    NodeB: x=270 y=65  w=110 h=50  → left-center=(270, 90)
    Arrow A→B horizontal: x1=210 y1=90  x2=262 y2=90 (shorten 8px for arrowhead)
  -->
  <line x1="210" y1="90" x2="262" y2="90"
        class="a-emphasis" stroke-width="1.5"
        marker-end="url(#arrowhead-emphasis)"/>

  <!-- 4. ALL NODES (mask + type rect + text) -->
  <!-- NodeA -->
  <rect x="100" y="65" width="110" height="50" rx="6" class="c-mask"/>
  <rect x="100" y="65" width="110" height="50" rx="6" class="c-backend" stroke-width="1.5"/>
  <text x="155" y="86" class="t-primary" font-size="11" font-weight="600" text-anchor="middle">Step 1</text>
  <text x="155" y="102" class="t-muted" font-size="9" text-anchor="middle">Description</text>

  <!-- NodeB -->
  <rect x="270" y="65" width="110" height="50" rx="6" class="c-mask"/>
  <rect x="270" y="65" width="110" height="50" rx="6" class="c-backend" stroke-width="1.5"/>
  <text x="325" y="86" class="t-primary" font-size="11" font-weight="600" text-anchor="middle">Step 2</text>
  <text x="325" y="102" class="t-muted" font-size="9" text-anchor="middle">Description</text>

  <!-- 5. Lane labels -->
  <text x="40" y="105" class="t-muted" font-size="9" font-weight="600"
        text-anchor="middle" transform="rotate(-90,40,105)">Lane 1</text>

  <!-- 6. Legend — gap ≥40px from lowest node bottom -->
  <!-- Lowest node bottom = 200, so legendY = 240 minimum -->
  <rect x="80" y="240" width="1000" height="56" rx="6" class="c-mask"/>
  <rect x="80" y="240" width="1000" height="56" rx="6" class="c-external" stroke-width="1"/>
  <rect x="100" y="254" width="14" height="10" rx="2" class="c-mask"/>
  <rect x="100" y="254" width="14" height="10" rx="2" class="c-backend" stroke-width="1"/>
  <text x="120" y="263" class="t-muted" font-size="8">Service</text>
  <!-- Footnote -->
  <text x="100" y="286" class="t-dim" font-size="7">Abbreviations and notes here</text>
</svg>
```

---

## Self-Review Checklist (v5.0)

Run ALL checks. All must pass before delivering.

**Quality Gate 1 — Arrow Connections:**
- [ ] For every arrow, verify x1/y1 equals exact edge midpoint of source box (computed, not estimated)
- [ ] For every arrow, verify x2/y2 equals exact edge midpoint of target box minus 8px arrowhead offset
- [ ] No arrow coordinate is a round-number guess (e.g., "150" when box center is 155)

**Quality Gate 2 — Uniform Node Sizing:**
- [ ] All nodes in the same swimlane or tier have identical `width` and `height`
- [ ] No node is visually wider or taller than its lane-peers without explicit justification

**Quality Gate 3 — Arrow Layering:**
- [ ] All `<line>` and `<path>` elements appear in SVG source BEFORE any `<rect class="c-mask">` node group
- [ ] No arrow appears after a node group in document order

**Quality Gate 4 — Legend Separation:**
- [ ] Legend top ≥ (lowest diagram element bottom + 40px)
- [ ] Legend has its own background panel (`c-mask` + `c-external` rect)
- [ ] viewBox height = legend bottom + 20px

**Quality Gate 5 — No Node Overlap:**
- [ ] For every pair of adjacent nodes in the same lane: (cx_B - w/2) - (cx_A + w/2) ≥ 16px
- [ ] For every pair of nodes in adjacent lanes at the same column: no vertical bounding box intersection
- [ ] No node's bounding box intersects any arrow path's visual midpoint

**Quality Gate 6 — Arrow Completeness & Shape Semantics:**
- [ ] Every arrow's (x1,y1) is on the edge of a named source node
- [ ] Every arrow's (x2,y2) is on the edge of a named target node — no dangling arrows
- [ ] START and END nodes use pill shape (rx = height/2), not plain rectangles
- [ ] Decision/branch nodes use diamond shape (`<polygon>`), not rectangles
- [ ] No arrow exists whose source or target node is not defined in the diagram

**Quality Gate 7 — Alignment & Equal Spacing:**
- [ ] All nodes in the same horizontal row share identical `cy`
- [ ] All nodes in the same vertical column share identical `cx`
- [ ] Spacing between consecutive nodes in a lane is equal (±2px tolerance)
- [ ] Primary flow direction is left→right within rows, top→bottom across rows
- [ ] No node is offset from its row/column grid without an explicit reason

**Quality Gate 8 — Breathing Room:**
- [ ] col_step = node_width + h_gap where h_gap ≥ 60px (recommended 80px)
- [ ] Every horizontal arrow visible length (gap − 8px offset) ≥ 40px
- [ ] Every vertical arrow visible length ≥ 40px
- [ ] Lane heights accommodate node_h + ≥56px padding (single row)

**General:**
- [ ] No hardcoded hex/rgba colors in SVG
- [ ] Every `c-<type>` rect has identical-geometry `c-mask` before it
- [ ] viewBox width/height exceed all element extents by ≥20px
- [ ] `.toolbar`, `<script>`, `:root`/`[data-theme]` CSS untouched
- [ ] Dark and light themes both render correctly


---

## Semantic Color Guide

| Component | Class | Examples |
|-----------|-------|---------|
| Frontend | `c-frontend` | Browser, Mobile, SPA, CDN |
| Backend | `c-backend` | API, Service, Worker, Lambda |
| Database | `c-database` | PostgreSQL, Redis, S3, MongoDB |
| Cloud | `c-cloud` | AWS/GCP/Azure services, Load Balancer |
| Security | `c-security` | Auth, JWT, WAF, Secrets |
| Message Bus | `c-messagebus` | Kafka, SQS, RabbitMQ |
| External | `c-external` | 3rd party APIs, generic, legend panels |
