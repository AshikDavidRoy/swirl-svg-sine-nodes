
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

### Data Preparation

```js
var data_array = data.concat();            // Deep copy of imported data
var finalized_array = { ... };             // Root tree with "start" > "about" > data
```

### Tree Parsing

```js
var tree = d3.layout.tree().size([width, height]);
var nodes = tree.nodes(data);
var links = tree.links(nodes);
```

D3 generates the basic tree, then we inject:

```js
// Extra swirl for links from nodes with `swirl`
links.forEach((link) => {
  if (link.source.swirl) link.swirl = link.source.swirl;
});
```

### Custom Links from `data.js`

```js
if (d.links) {
  d.links.forEach((link) => {
    const fromNode = nodes.find(n => n.name === link.fromNode);
    const toNode = nodes.find(n => n.name === link.toNode);
    links.push({ source: fromNode, target: toNode, swirl: link.swirl });
  });
}
```

### Swirl Path Generator

```js
function generateSwirlPath(d) {
  const swirl = d.swirl || defaultSwirl;
  const offset = swirl.amplitude * Math.sin(frequency * y + phase);
  return `M... C... C...`;
}
```

Every link is rendered as a curved Bézier path using that math.

### Goal Links

```js
if (d.toGoal) {
  links.push({ source: d, target: GoalNode, swirl: d.swirl });
}
```

Adds final curved paths from any `toGoal` node to the `"Goal"`.

---

## 🛠️ To Customize

1. **Edit `data.js`**

   * Add or remove nodes
   * Control swirls, positions, and cross-links

2. **Swirl Example**

```js
swirl: {
  direction: 1,
  amplitude: 120,
  frequency: 0.03,
  phase: 0.5
}
```

3. **Add Cross Links**

```js
links: [
  {
    fromNode: "root 1-2",
    toNode: "root 3-2",
    swirl: { direction: -1, amplitude: 140, frequency: 0.02, phase: 1.2 }
  }
]
```

---
