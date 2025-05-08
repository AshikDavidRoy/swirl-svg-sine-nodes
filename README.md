# 🌪️ Swirl Path Graph: Interactive Node-Link Tree with Curved Connections

This project visualizes a dynamic **tree-based node-link diagram** using [D3.js](https://d3js.org/) with **customizable swirl paths** (curved connections) between nodes. These swirl paths are controlled entirely via `data.js`, making it easy to define how each node connects to others using animated or organic-looking curves.

---

## 🚀 Features

- Hierarchical node structure rendered using D3’s **tree layout**
- Swirl paths based on the **sine wave function**
- Fully **customizable per-link behavior** (direction, amplitude, frequency, phase)
- Support for **cross-branch links** using `fromNode` → `toNode` pairs
- Interactive node highlighting and responsive SVG resizing

---

## 📁 File Structure

```

/project-root
│
├── index.html       # Basic HTML scaffold with SVG container
├── data.js          # Defines all nodes, links, and swirl curve parameters
├── graph.js         # Builds the graph using D3.js and renders swirled paths
└── README.md        # This file

````

---

## 🔢 Algorithms & Layout Logic

### 1. **Tree Layout** (via D3.js)

We use the `d3.layout.tree()` layout engine to generate a vertical tree structure. It computes:

- `d.x`: horizontal position
- `d.y`: vertical position (based on `depth`)

These are adjusted further for aesthetic spacing.

---

### 2. **Swirl Path Generation** (Math Behind the Curves)

Every curved connection between nodes is drawn using a custom cubic Bézier curve with a sine wave offset:

```js
const swirlOffsetX = direction * amplitude * Math.sin(frequency * y + phase);
````

#### 🔍 Parameters:

| Parameter   | Description                                        |
| ----------- | -------------------------------------------------- |
| `direction` | Swirl direction (1 for rightward, -1 for leftward) |
| `amplitude` | How far the curve swings horizontally              |
| `frequency` | How fast the sine wave oscillates vertically       |
| `phase`     | Where the curve starts in its wave cycle           |

This creates an organic “swirl” in the X-direction based on the Y-position of the node.

---

## ⚙️ Customization via `data.js`

Every node can have the following:

```js
{
  name: "root 1-2",
  path: "root 1",
  toGoal: true,
  swirl: {
    direction: -1,
    amplitude: 100,
    frequency: 0.02,
    phase: 1.0
  },
  xOffset: -50,
  yOffset: 20,
  links: [
    {
      fromNode: "root 1-2",
      toNode: "root 3-2",
      swirl: { direction: 1, amplitude: 150, frequency: 0.05, phase: 0.8 }
    }
  ]
}
```

### 🧠 Custom Behavior:

* `swirl`: affects how the link from this node to its child (or the goal) will curve.
* `xOffset` / `yOffset`: custom positions for fine-tuning where the node appears.
* `toGoal`: if `true`, automatically draws a swirl link from this node to the `"Goal"` node.
* `links[]`: defines custom non-tree connections using `fromNode` and `toNode`.

---

## 📜 How `graph.js` Works — Line by Line Summary
# **Tornado Tree Graph: Technical Explanation**

This document explains how the **Tornado Tree Graph** works, focusing on:
1. **Data Structure & Tree Construction**
2. **Swirling Path Algorithm (Sine Wave-Based Links)**
3. **D3.js Visualization Pipeline**
4. **Interactive Features (Highlighting & Resizing)**

---

## **1. Data Structure & Tree Construction**
The graph is built from a **nested hierarchical dataset** (`data.js`) structured as follows:

### **Node Properties**
| Property | Description |
|----------|------------|
| `name` | Unique identifier for the node |
| `path` | Grouping key (used for highlighting) |
| `toGoal` | If `true`, connects directly to the "Goal" node |
| `xOffset`, `yOffset` | Adjusts node position from default layout |
| `swirl` | Controls the curvature of outgoing links |
| `children` | Array of child nodes (recursive structure) |
| `links` | Additional cross-connections between nodes |

### **Tree Modifications**
- The `pathGraph()` function:
  - Adds a **"Goal" node** to all terminal nodes (leaf nodes).
  - Collects all parent nodes that need direct connections to the goal (`parent_array`).
  - Prepares the data for D3.js processing.

---

## **2. Swirling Path Algorithm (Sine Wave-Based Links)**
The **key innovation** in this visualization is the **swirling Bézier curves** connecting nodes, generated using **sine wave modulation**.

### **Swirl Configuration**
Each link can define a `swirl` object:
```js
swirl: {
  direction: 1,       // 1 (right) or -1 (left)
  amplitude: 100,     // Wave "strength" (width of curve)
  frequency: 0.03,    // How fast the wave oscillates
  phase: 0.5          // Shifts the wave along the y-axis
}
```

### **Path Generation (`generateSwirlPath()`)**
The function generates a **cubic Bézier curve** (`C` command in SVG paths) with a **sinusoidal offset**:
```js
function generateSwirlPath(d) {
  const { source, target } = d;
  const curveHeight = (target.y - source.y) / 2;

  const swirl = d.swirl || { direction: 1, amplitude: 80, frequency: 0.02, phase: 0 };
  const direction = swirl.direction;
  const amplitude = swirl.amplitude;
  const frequency = swirl.frequency;
  const phase = swirl.phase;

  // Apply sine wave to control point X-coordinates
  const swirlOffsetX = direction * amplitude * Math.sin(frequency * source.y + phase);

  return `
    M${source.x},${source.y}
    C${source.x + swirlOffsetX},${source.y + curveHeight},
     ${target.x - swirlOffsetX},${target.y - curveHeight},
     ${target.x},${target.y}
  `;
}
```

#### **How It Works**
1. **`M${source.x},${source.y}`**  
   - Moves to the **source node**.
2. **`C${x1},${y1}, ${x2},${y2}, ${x3},${y3}`**  
   - **Control points (`x1, y1` and `x2, y2`)** define the curve’s shape.
   - **`swirlOffsetX`** applies a **sine wave** to shift the control points:
     - `Math.sin(frequency * y + phase)` creates oscillations.
     - `amplitude` scales the effect.
     - `direction` flips the curve left/right.
3. **Final point (`x3, y3`)** lands on the **target node**.

### **Visual Effect**
- The **frequency** controls how many waves appear along the link.
- The **amplitude** controls how wide the curve swings.
- The **phase** shifts the wave along the y-axis.

---

## **3. D3.js Visualization Pipeline**
The graph is rendered using **D3.js (v3)** with the following steps:

### **A. Tree Layout Setup**
```js
const tree = d3.layout.tree().size([graphWidth, graphHeight]);
```
- Computes **node positions** in a hierarchical layout.

### **B. Node Positioning**
- **Default D3.js Tree Layout:**  
  - Nodes are spaced vertically (`y = depth * 200`).
  - Horizontally centered (`x = (x - graphWidth / 2) * (1 - depth * 0.1)`).
- **Custom Offsets:**  
  - `xOffset` and `yOffset` adjust positions.

### **C. Link Generation**
- **Standard Links:** Extracted via `tree.links(nodes)`.
- **Extra Links:** Added from `node.links` (cross-connections).
- **Goal Links:** Added for nodes with `toGoal: true`.

### **D. Rendering**
- **Nodes:** Rendered as circles with labels.
- **Links:** Drawn as **swirling paths** using `generateSwirlPath()`.

---

## **4. Interactive Features**
### **Highlighting Paths**
- Clicking a node **highlights all nodes** with the same `path`.
- Implemented via jQuery:
  ```js
  $(".path-graph").on("click", "svg .node[path]", function () {
    var path = $(this).attr("path");
    $("svg [path]").removeClass("highlight");
    $("svg").find("[path='" + path + "']").addClass("highlight");
  });
  ```

### **Responsive Resizing**
- The graph **redraws on window resize**:
  ```js
  d3.select(window).on("resize", function () {
    var selected_path = $(".node.highlight[path]").attr("path");
    d3.select(".path-graph svg g").remove();
    generate_graph(finalized_array, parent_array);
    $("svg").find("[path='" + selected_path + "']").addClass("highlight");
  });
  ```

---

## **Conclusion**
This visualization combines:
✅ **Hierarchical tree layout** (D3.js)  
✅ **Sine-wave-based swirling links** (custom Bézier curves)  
✅ **Interactive path highlighting** (jQuery/D3.js)  
✅ **Responsive design** (auto-resizing)  

The **swirling effect** is mathematically controlled by **amplitude, frequency, and phase**, making it highly customizable. This approach could be extended for **flow diagrams, decision trees, or network graphs** requiring organic, curved connections.
