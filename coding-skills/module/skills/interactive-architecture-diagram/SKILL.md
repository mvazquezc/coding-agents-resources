---
name: interactive-architecture-diagram
description: Build interactive single-file HTML architecture diagrams with clickable connection views, animated request flow visualizations, dark mode, and responsive layout. Use when creating infrastructure diagrams, cluster topologies, multi-tier architecture views, or request routing animations.
---

# Interactive Architecture Diagram

Build self-contained single-file HTML diagrams that visualize infrastructure architecture with interactive connection exploration and animated request flows.

## Architecture

Everything lives in one `.html` file: CSS, HTML, and JS together. No external dependencies.

### Core Pattern: Tab System

Use a CSS slider for tabs (no JS framework needed):

```css
.tab-slider { display:flex; width:300%; /* N * 100% for N tabs */ }
.tab-panel { width:33.333%; flex-shrink:0 }
.tab-slider.at-1 { transform:translateX(-33.333%) }
.tab-slider.at-2 { transform:translateX(-66.667%) }
```

```js
function switchTab(i) {
  document.querySelectorAll('.tab-btn').forEach(function(b,j) {
    b.classList.toggle('active', j===i);
  });
  var sl = document.getElementById('tabSlider');
  sl.className = 'tab-slider' + (i > 0 ? ' at-' + i : '');
}
```

### Typical Tab Types

1. **Cluster Location** -- static inventory of clusters grouped by provider/location
2. **Grid Architecture** -- interactive component boxes with clickable connection views
3. **Flow Visualization** -- animated request routing with SVG `<animateMotion>`

---

## Component Boxes

Use nested CSS classes for the box hierarchy:

| Class | Purpose | Nesting |
|-------|---------|---------|
| `.aregion` | Top-level region (Cloud, On-Prem) | Contains `.ainner` |
| `.ainner` | Cluster or sub-group box | Contains `.achip` |
| `.achip` | Component chip (RHOAI, MaaS, etc.) | Leaf |
| `.achip.model` | Model chip (M1, M2, etc.) | Leaf, muted style |
| `.abox` | Standalone box (Git Repo, etc.) | Self-contained |

Always add `id` attributes to any element that will be an arrow target:

```html
<div id="nd-mgmt" class="ainner connectable" onclick="event.stopPropagation();showConnView('nd-mgmt')">
  <div class="ainner-title">AI Grid Control Plane</div>
  <div class="ainner-row">
    <div class="achip">RHACM</div>
    <div id="nd-mgmt-maas" class="achip">MaaS</div>
  </div>
</div>
```

**ID naming convention**: `nd-` prefix for node-level targets, sub-elements use `nd-{parent}-{component}` (e.g., `nd-l40-maas`, `nd-aws-m1`).

---

## Glow Effect for Connectable Boxes

```css
@keyframes connGlow {
  0%,100% { box-shadow:0 0 8px 2px rgba(224,49,49,.15) }
  50% { box-shadow:0 0 18px 5px rgba(224,49,49,.4) }
}
.connectable { animation:connGlow 2.5s ease-in-out infinite; position:relative }
.connectable:hover { animation:none; box-shadow:0 0 22px 6px rgba(224,49,49,.5)!important }
```

Add a tooltip via `::after` pseudo-element: "Click to view connections".

---

## Connection View (Left Panel + Arrows)

### Architecture

When a connectable box is clicked:
1. A **fixed-position left panel** (`#connPanel`) slides in showing the box name and clickable sub-components
2. A **fixed-position SVG overlay** (`#connSvg`) draws arrows from the panel to target elements
3. A **backdrop** (`#connBackdrop`) dims the rest and catches clicks to dismiss

**Critical**: Place the panel, backdrop, and SVG at **body level** (outside any CSS-transformed containers like tab sliders). `position:fixed` breaks inside `transform` parents.

```html
<!-- At body level, NOT inside tab-slider -->
<div id="connBackdrop" onclick="hideConnView()"></div>
<div id="connPanel">
  <div style="background:var(--rh-red);padding:14px 18px">
    <div id="connTitle"></div>
    <div id="connSubtitle"></div>
  </div>
  <div id="connChips"></div>
</div>
<svg id="connSvg">
  <defs><marker id="cah" ...></defs>
</svg>
```

### Connection Data Structure

Two types of entries:

**Component-based** (clusters): user must click a sub-component to see its arrows.

```js
'nd-mgmt': {
  label: 'AI Grid Control Plane',
  info: 'arch-mgmt',           // ID for the detail modal
  components: {
    'RHACM': { targets: [
      {id:'nd-git', label:'Source of Truth (GitOps)'},
      {id:'nd-aws', label:'Configures / Deploys'}
    ]},
    'MaaS': { targets: [
      {id:'nd-llmd-gw', label:'Exposes'}
    ]}
  }
}
```

**Direct targets** (personas): arrows draw immediately on click.

```js
'persona-admin': {
  label: 'AI Grid Admin',
  desc: 'Description shown in the panel body.',
  targets: [
    {id:'nd-git', label:'Describes IaaC and CaaC'},
    {id:'nd-mgmt', label:'Manages and Controls'}
  ]
}
```

### Arrow Drawing: Trunk-and-Fork Pattern

When multiple arrows share the same label text, draw them as:

```
                    ┌─────────────────┐
origin ──────────── │  Label Text     │ ──── Target 1
                    └─────────────────┘ ──── Target 2
                                        ──── Target 3
```

1. **Group** targets by label text
2. **Trunk**: short straight line from the panel's active chip to the left edge of a label pill
3. **Pill**: white rounded rect with colored border containing the label text (one per unique label)
4. **Branches**: curved Bezier paths from the pill's right edge to each target
5. **Space** pills vertically (min 28px) to prevent overlap

The fork Y position blends 25% toward the average target Y:
```js
var forkY = oy + (avgTy - oy) * 0.25;
```

### Scroll Handling

Store the current targets and redraw on scroll:

```js
var connCurrentTargets = null;
window.addEventListener('scroll', function() {
  if (connActive && connCurrentTargets) drawConnArrows(connCurrentTargets);
}, true); // capture phase to catch all scroll containers
```

---

## Persona Icons

Add persona icons (Admin, Tenant Admin, Users) as clickable cards above the architecture diagram. They use direct targets (no sub-components).

```html
<div class="persona-btn connectable" onclick="event.stopPropagation();showConnView('persona-admin')">
  <svg viewBox="0 0 24 24" fill="currentColor"><!-- person icon --></svg>
  <span>AI Grid Admin</span>
</div>
```

Descriptions appear in the left panel body (not on the icon itself).

---

## Flow Visualization (Animated Request Routing)

### SVG Helper Functions

Build flow diagrams programmatically with these helpers:

```js
function svgNode(x, y, w, h, label, sub, cls)  // Renders a positioned box
function svgArrow(id, path, color, dashed)       // Renders a named SVG path
function svgDot(id, pathId, color, dur, beginRef) // Animated dot along a path
function svgMarker(color)                         // Arrow head marker
function svgLabel(x, y, text, color)              // Text label with halo
```

### Sequential Animation (One Ball at a Time)

Use SVG `<animateMotion>` with chained `begin` references:

```js
function svgDot(id, pathId, color, dur, beginRef) {
  return '<circle r="5" fill="'+color+'" opacity="0">'+
    '<animateMotion id="'+id+'" dur="'+dur+'s" begin="'+beginRef+'" fill="freeze">'+
      '<mpath href="#'+pathId+'"/>'+
    '</animateMotion>'+
    '<animate attributeName="opacity" values="0;1;1;0" keyTimes="0;0.01;0.9;1" dur="'+dur+'s" begin="'+id+'.begin" fill="freeze"/>'+
  '</circle>';
}
```

Chain them: first dot starts at `"0s;lastDot.end+1.5s"` (loop), subsequent dots start at `"prevDot.end"`.

### Flow Scenarios

For complex flows (failover, session stickiness), use sub-tabs that re-render the SVG:

```js
function renderFlow3(container, scenario) {
  // scenario 0 = session stickiness, 1 = failover
  // Rebuild entire SVG with different paths/animations per scenario
}
```

**Failover animation**: Use `<animate>` on broken-link X marks with timed `keyTimes` matching the animation cycle:

```js
s += '<line ... stroke="red" opacity="0">';
s += '<animate attributeName="opacity" values="0;0;1;1" keyTimes="0;0.33;0.37;1" dur="20s" repeatCount="indefinite"/>';
s += '</line>';
```

### Color Conventions

| Element | Color | Usage |
|---------|-------|-------|
| Forward request arrows | `#228be6` (blue) | Request path |
| Return response arrows | `#2b8a3e` (green, dashed) | Response path |
| Fail/broken | `#e03131` (red) | Broken links |
| User A ball | `#868e96` (gray) | First user's request |
| User B ball | `#e8590c` (orange) | Second user's request |
| Connection arrows | `#e03131` (red) | Architecture tab connections |

---

## Dark Mode

### CSS Variable Override

```css
[data-theme="dark"] {
  --bg:#121417; --card-bg:#1e2025; --border:#3a3f47;
  --text:#e4e5e7; --text-muted:#a0a4ab;
}
```

### Theme Toggle

Sun/moon icon button in the tab bar. Moon shows in light mode (click to go dark), sun shows in dark mode (click to go light). Save preference to `localStorage`.

### JS-Rendered SVGs Must Be Theme-Aware

Add an `isDark()` helper and use it in every SVG rendering function:

```js
function isDark() { return document.documentElement.getAttribute('data-theme') === 'dark' }
```

**Critical colors to adapt in dark mode:**
- Node fills: `#fff` -> `#1e2025`
- Node text: `#1e1e1e` -> `#e4e5e7`
- Sub-text: `#6c757d` -> `#a0a4ab`
- Label halos: `stroke:#fff` -> `stroke:#1e2025`
- Connection pill background: `#fff` -> `#1e2025`
- Connection pill text: `#e03131` -> `#ff6b6b`

**Re-render SVGs on theme toggle** if the user is on a tab with JS-rendered content.

### Common Dark Mode CSS Gaps to Watch

These hardcoded light colors need explicit `[data-theme="dark"]` overrides:
- `.cluster-node.active { background:#fff3f3 }` -- light pink
- `.achip:hover { background:#f1f3f5 }` -- light gray hover
- `.achip.dashed`, `.abox.dashed` -- `border-color:#adb5bd` too dim
- `.legend-swatch { border:rgba(0,0,0,.1) }` -- invisible on dark

---

## Responsive Design

Architecture tab grids use inline `style` for `grid-template-columns`. Add CSS classes alongside so media queries can override:

```html
<div class="arch-inf-grid" style="display:grid;grid-template-columns:1fr 1fr 1fr;gap:14px">
```

```css
@media(max-width:900px) {
  .arch-cloud-grid { grid-template-columns:1fr!important }
  .arch-inf-grid { grid-template-columns:1fr 1fr!important }
}
@media(max-width:600px) {
  .arch-inf-grid { grid-template-columns:1fr!important }
  .arch-tenant-grid { grid-template-columns:1fr!important }
  #connPanel { width:200px; left:10px }
  .connectable::after { display:none } /* hide tooltips on mobile */
}
```

---

## Detail Modal

Use a backdrop + centered panel for component details on click:

```js
var D = {
  'arch-mgmt': {
    t: 'AI Grid Control Plane',     // title
    c: '#e03131',                    // swatch color
    d: [['Location','On-Prem']],     // detail rows [key, value]
    comp: [{n:'RHACM'}, {n:'MaaS', o:1}]  // components, o:1 = optional/dashed
  }
};
```

---

## Checklist for New Diagrams

1. Define all boxes with proper `id` attributes for anything that will be an arrow target
2. Define `CONNS` data: component-based for clusters, direct targets for personas
3. Add `connectable` class and `showConnView()` onclick to interactive boxes
4. Add persona icons if the architecture has distinct user roles
5. For flow tabs: define flows as JS render functions using the SVG helpers
6. Add `isDark()` checks in all SVG rendering functions
7. Add responsive CSS classes alongside inline grid styles
8. Add scroll listener for connection arrow redrawing
9. Test dark mode toggle re-renders JS SVG content
10. Validate: div/svg tag balance, JS syntax check with `node --check`
