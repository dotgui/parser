# gui-parser

Parses a `.gui` XML string into a plain, JSON-serializable object tree.

No DOM output. No rendering. Just data.

---

## What it does

Takes a `.gui` markup string and returns a structured object with:

- **`root`** — the full UI tree as nested `{ type, children, ...attrs }` nodes
- **`tokens`** — design tokens as a flat key/value map
- **`fonts`** — font declarations
- **`styles`** — named text styles
- **`fillStyles`** — named fill (color) styles
- **`effectStyles`** — named effect styles
- **`assets`** — resolved asset map `{ '$img-1': 'data:image/webp;base64,...' }`
- **`version`** / **`name`** — document metadata

---

## Usage

```ts
import { parse } from 'gui-parser'

const result = parse(guiXmlString)

if (result) {
  console.log(result.root)     // the UI tree
  console.log(result.tokens)   // { primary: '#007AFF', ... }
}
```

With a pre-built asset map (skips re-parsing base64 blobs):

```ts
const result = parse(guiXmlString, assetMap)
```

Returns `null` if the XML is invalid.

---

## Node shape

Every node in the tree follows the same shape:

```ts
{
  type: string        // the tag name: "frame", "stack", "text", "shape", etc.
  children?: Node[]   // child layout/content nodes
  segments?: Node[]   // text segments (mixed-style text only)
  paths?: string[]    // SVG path data (shape nodes only)
  appearance?: {      // multi-fill or complex effect block
    fills: Node[]
    effects: Node[]
  }
  // ...all other attributes as properties
}
```

### Example

```json
{
  "type": "stack",
  "direction": "vertical",
  "gap": 16,
  "padding": "24",
  "fill": "#F2F2F7",
  "children": [
    {
      "type": "text",
      "value": "Hello, dotgui",
      "font-family": "Inter",
      "font-size": 22,
      "font-weight": 700,
      "color": "#1C1C1E"
    },
    {
      "type": "shape",
      "shape-type": "rect",
      "width": 342,
      "height": 50,
      "fill": "$primary",
      "radius": 12
    }
  ]
}
```

---

## Value types

| Value | Output type |
|---|---|
| Known numeric attributes (`width`, `height`, `gap`, `font-size`, `opacity`, ...) | `number` |
| `"true"` / `"false"` | `boolean` |
| Hex colors (`#1C1C1E`) | `string` |
| Token references (`$primary`) | `string` |
| Functional values (`linear-gradient(...)`, `clamp(...)`) | `string` — unparsed |
| Everything else | `string` |

Functional values like `linear-gradient()` are intentionally kept as strings. Each function type will be handled by a dedicated value parser in a future release when the full function vocabulary is defined.

---

## Who uses this

- **AI agents** — read the UI tree as structured data without a browser
- **Code generators** — walk the tree to emit React, SwiftUI, or any target
- **Tooling** — diff, lint, or transform `.gui` files programmatically

`gui-render` is the sibling package that takes the same `.gui` string and renders it to a live DOM. They are independent — neither depends on the other.
