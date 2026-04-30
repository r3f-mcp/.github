# r3f-mcp

**Give AI eyes and hands inside your 3D scene.**

An MCP (Model Context Protocol) server that connects AI coding tools (Claude, Cursor, Windsurf, VS Code Copilot) to a running React Three Fiber scene. Read the scene graph, edit objects, preview changes in real time, all through natural language.

Blender has an MCP server. Figma has one. The 3D web didn't, until now.

---

## Why this exists

AI coding tools are powerful but blind when it comes to 3D. Ask Claude or Cursor to build a Three.js scene and it's writing code it can't see. It's guessing at positions, materials, and lighting without any feedback loop. The result is hallucinated APIs, broken layouts, and endless trial-and-error.

r3f-mcp closes that gap. It gives AI agents the ability to:

- **See** the live scene graph (every mesh, light, camera, group, and their properties)
- **Edit** objects in real time (transform, material, geometry, visibility — reflected instantly in the browser)
- **Preview** the result before committing changes to code
- **Query** scene state ("what's the bounding box of this group?", "which meshes use this material?", "what's visible in the current camera frustum?")
- **Scaffold** new R3F components from scene context (the AI knows what's already in the scene when generating new code)

Think of it as React DevTools for Three.js, but the inspector is an AI agent.

---

## Quick start

### Install

```bash
npm install r3f-mcp
```

### Add the provider to your R3F app

```jsx
import { Canvas } from '@react-three/fiber'
import { MCPProvider } from 'r3f-mcp'

function App() {
  return (
    <Canvas>
      <MCPProvider port={3333}>
        <ambientLight intensity={0.5} />
        <mesh position={[0, 1, 0]}>
          <boxGeometry args={[1, 1, 1]} />
          <meshStandardMaterial color="hotpink" />
        </mesh>
      </MCPProvider>
    </Canvas>
  )
}
```

### Connect your AI tool

**Claude Desktop:** Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "r3f": {
      "command": "npx",
      "args": ["r3f-mcp", "--port", "3333"]
    }
  }
}
```

**Cursor / Windsurf:** Add to your MCP configuration:

```json
{
  "r3f": {
    "command": "npx",
    "args": ["r3f-mcp", "--port", "3333"]
  }
}
```

That's it. Your AI agent can now see and interact with your R3F scene.

---

## What the AI can do

### Read the scene

```
"What objects are in the scene?"
"Show me the properties of the red cube"
"What materials are being used?"
"What's the camera position?"
```

The AI receives a structured representation of your entire scene graph — objects, transforms, geometries, materials, lights, cameras — updated in real time.

### Edit objects

```
"Move the cube to position [2, 0, 0]"
"Change the sphere's color to blue"
"Scale the entire group by 1.5"
"Hide the debug wireframe"
"Set the directional light intensity to 0.8"
```

Changes are applied instantly in the running scene. The AI sees the result and can iterate.

### Query spatial relationships

```
"What's inside the bounding box of the player group?"
"Which objects are within 5 units of the camera?"
"What's the distance between the two meshes?"
"Is this object visible from the current camera angle?"
```

### Generate code from context

```
"Add a spotlight pointing at the main character"
"Create a particle system around the existing sphere"
"Build a floor grid that matches the scene scale"
```

Because the AI knows what's already in the scene — positions, scales, materials, naming conventions — the generated R3F code is contextually aware, not generic boilerplate.

---

## Available tools

| Tool | Description |
|------|-------------|
| `scene_graph` | Returns the full scene tree with transforms, geometries, materials |
| `get_object` | Get detailed properties of a specific object by name or uuid |
| `set_transform` | Update position, rotation, or scale of an object |
| `set_material` | Update material properties (color, opacity, metalness, roughness, etc.) |
| `set_visible` | Toggle visibility of an object or group |
| `query_bounds` | Get bounding box of an object or group |
| `query_distance` | Measure distance between two objects |
| `query_frustum` | List objects visible in the current camera frustum |
| `screenshot` | Capture the current rendered frame as an image |
| `add_object` | Insert a new primitive (mesh, light, group) into the scene |
| `remove_object` | Remove an object from the scene |

---

## How it works

r3f-mcp has two parts:

1. **A React component (`<MCPProvider>`)** that wraps your R3F scene and exposes the Three.js scene graph over a local WebSocket connection. It hooks into R3F's reconciler to stay in sync with React state.

2. **An MCP server** that translates between the Model Context Protocol (what AI tools speak) and the WebSocket connection to your scene. The server registers tools that the AI agent can call, and routes responses back.

```
┌─────────────┐     MCP      ┌─────────────┐   WebSocket   ┌──────────────┐
│  Claude /    │◄────────────►│  r3f-mcp    │◄─────────────►│  Your R3F    │
│  Cursor /    │   (stdio)    │  server     │   (localhost)  │  app in      │
│  Windsurf    │              │             │                │  browser     │
└─────────────┘              └─────────────┘               └──────────────┘
```

The provider is read-write: edits from the AI are applied to the Three.js scene directly and optionally synced back to your React component state via a callback prop.

---

## Configuration

```jsx
<MCPProvider
  port={3333}              // WebSocket port (default: 3333)
  readOnly={false}         // Set true to disable write operations
  onEdit={(edit) => {}}    // Callback when AI makes an edit — use this to sync back to React state
  include={['meshes']}     // Filter what's exposed (default: everything)
  exclude={['helpers']}    // Exclude specific object types or names
  screenshotQuality={0.8}  // JPEG quality for screenshot tool (0-1)
/>
```

---

## Works with

- **React Three Fiber** v8+
- **Three.js** r150+
- **@react-three/drei** — all drei components are visible in the scene graph
- **@react-three/rapier** — physics bodies appear with their collider shapes
- **@react-three/postprocessing** — effects chain is queryable
- **Any MCP-compatible AI tool** — Claude Desktop, Cursor, Windsurf, VS Code + Copilot, and any future MCP client

---

## Roadmap

**v0.1 (current)**
- Scene graph reading
- Basic transforms (position, rotation, scale)
- Material property editing
- Object visibility toggle
- Screenshot capture

**v0.2**
- Add/remove objects
- Spatial queries (bounds, distance, frustum)
- Scene diffing (what changed since last query)

**v0.3**
- Animation timeline inspection
- Physics state reading (Rapier integration)
- Performance metrics (draw calls, triangles, FPS)

**v0.4**
- Code generation with scene context
- Component scaffolding from natural language
- Shader/material preview and editing

**Future**
- WebGPU / TSL support
- Multi-scene support
- Collaborative editing (multiple AI agents)
- Visual regression testing

---

## Philosophy

This project believes that the best creative tools start with understanding how people actually work, not how computers think they should. 

3D development on the web is already hard. R3F made it dramatically more accessible by bringing it into React's mental model. r3f-mcp takes the next step: making AI a genuine collaborator in the creative process, not just a code generator working in the dark.

Engineering starts with people, not code.

---

## Contributing

This project is in early development. Issues, PRs, and ideas are welcome.

If you're building something with r3f-mcp, I'd love to hear about it! Reach out on [X](https://x.com/louannemmurphy) or [Bluesky]([https://bsky.app/profile/louanne.me](https://bsky.app/profile/louannemur.bsky.social)).

---

## License

MIT
