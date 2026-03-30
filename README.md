<p align="center">
  <img src="https://img.shields.io/badge/Unity-2021.3%2B-black?logo=unity" alt="Unity 2021.3+"/>
  <img src="https://img.shields.io/badge/License-MIT-green" alt="MIT License"/>
  <img src="https://img.shields.io/badge/MCP-Compatible-blue" alt="MCP Compatible"/>
  <img src="https://img.shields.io/badge/Tools-20-orange" alt="20 Tools"/>
  <img src="https://img.shields.io/badge/C%23_Lines-5.7k-blueviolet" alt="5.7k Lines of C#"/>
  <img src="https://img.shields.io/badge/CPU_When_Idle-0%25-brightgreen" alt="0% CPU When Idle"/>
</p>

# Unity x Claude

> **Control Unity with natural language.** Ask Claude to create scripts, build scenes, tweak settings, or kick off builds — it talks directly to the Unity Editor.

This plugin adds an MCP server inside Unity so Claude can see and modify everything in your project. It uses zero CPU when idle and responds instantly when Claude sends a command.

---

## Setup (3 minutes)

### Step 1 — Add the plugin to your Unity project

Download or clone this repo, then copy the files into your project's `Packages` folder:

```
YourProject/
  Packages/
    com.claude.unity-mcp/       ← put everything here
      Editor/
      package.json
      mcp-bridge.mjs
```

Open Unity. You should see this in the console:

```
[MCP] Ready on port 9999
```

You can also check **Window > Claude MCP** — it shows the server status.

### Step 2 — Tell Claude where to find Unity

You need to add a few lines to Claude's config file so it knows how to connect.

**Find your config file:**

| OS | Location |
|---|---|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |

**Add this to the file** (create it if it doesn't exist):

```json
{
  "mcpServers": {
    "unity": {
      "command": "node",
      "args": [
        "/full/path/to/YourProject/Packages/com.claude.unity-mcp/mcp-bridge.mjs"
      ]
    }
  }
}
```

> **💡 Easy way:** In Unity, go to **Window > Claude MCP** and click **Copy Config**. It gives you the JSON with the correct path already filled in. Just paste it into the config file.

**Using Claude Code or Cursor?** These support direct HTTP — just point to:
```
http://localhost:9999/mcp
```

### Step 3 — Restart Claude and test

Restart Claude Desktop (or start a new chat). Then ask Claude something like:

- *"What's in my Unity scene?"*
- *"Create a red cube at position 0, 3, 0"*
- *"Add a Rigidbody to the player"*

If Claude responds with your scene data, you're connected. That's it!

---

## What can Claude do?

Once connected, Claude has **20 tools** to control Unity. Here's what you can ask for:

### 🎮 Scene & Objects
- Create, delete, duplicate GameObjects
- Read the full scene hierarchy
- Set transforms, tags, layers

### 🧩 Components
- Add/remove any component (Rigidbody, Collider, AudioSource, etc.)
- Read and modify any property on any component
- Batch-update multiple properties at once

### 📝 Scripts
- Create new C# scripts (MonoBehaviour, ScriptableObject, etc.)
- Modify existing scripts — find/replace, add methods, full rewrites

### 📦 Assets
- Search assets by name or type
- Create Materials, Prefabs, ScriptableObjects, Folders
- Import and re-import assets

### ⚙️ Editor & Settings
- Play, pause, stop, save, undo, redo
- Read and change project settings (Physics, Quality, Player, etc.)
- Control the Scene View camera

### 🔨 Build & Packages
- Build for any platform (Windows, macOS, WebGL, Android, iOS)
- Add/remove Unity packages
- Read console logs and errors

### 💻 Code Execution
- Run arbitrary C# code inside the editor — full access to UnityEngine and UnityEditor APIs

---

## Docs for Power Users

| Guide | What's in it |
|-------|-------------|
| **[SKILL.md](SKILL.md)** | Tool reference, common workflows, property path cheatsheet, depth strategy tips |
| **[WORKFLOW.md](WORKFLOW.md)** | Architecture deep-dive, known gotchas + workarounds, runtime testing pipeline, screenshot verification, domain reload handling |

### Use as a Claude Skill

For the best experience, copy `SKILL.md` into your Claude skills folder:

```
~/.claude/skills/unity-x-claude/SKILL.md
```

This gives Claude permanent access to the full tool reference, so it always knows exactly how to use every tool without you having to explain.

---

## Troubleshooting

**Claude says "Cannot connect to Unity"**
→ Make sure Unity is open and **Window > Claude MCP** shows "Running"
→ Check that nothing else is using port 9999

**Server not starting**
→ Open **Window > Claude MCP** and make sure the toggle is enabled
→ It auto-starts on every domain reload — try recompiling (Ctrl+R)

**Wrong project path**
→ The path in your Claude config must point to the **currently open** Unity project
→ Use **Window > Claude MCP > Copy Config** to get the correct path

**Node.js not found**
→ Make sure Node.js is installed: `node --version` in terminal
→ The bridge script (`mcp-bridge.mjs`) needs Node to translate between Claude and Unity

**Port conflict**
→ Go to **Window > Claude MCP > Advanced Settings** and change the port
→ Update your Claude config to match

---

## How it works

```
Claude Desktop / Claude Code / Cursor
        |
        | stdio (JSON-RPC)
        v
  mcp-bridge.mjs          ← Node.js translates between Claude and Unity
        |
        | HTTP to localhost:9999
        v
  Unity Editor             ← MCP server runs inside the editor process
        |
        |── 20 tool handlers (scripts, scenes, components, assets, etc.)
        |── Background thread with Socket.Poll (zero CPU when idle)
        |── Main thread dispatcher (safe Unity API access)
```

The server sleeps until Claude sends a request. No polling loops, no timers, no background tasks eating CPU.

---

## File Structure

```
com.claude.unity-mcp/
  package.json              Unity package manifest
  mcp-bridge.mjs            Node.js stdio-to-HTTP bridge
  Editor/
    MCPServer.cs             Main server — start/stop/restart, tool routing
    Communication/
      StreamableHttpServer   TCP listener (single thread, Socket.Poll)
      MainThreadDispatcher   Background → main thread queue
      JsonRpcHandler         JSON-RPC 2.0 parsing
      MiniJson               Lightweight JSON (zero dependencies)
    Tools/
      ScriptTools            Create / modify C# scripts
      SceneTools             Scene hierarchy operations
      ComponentTools         Component CRUD + runtime data
      AssetTools             Asset search / create / import
      EditorTools            Editor commands, selection, scene view
      SettingsTools          Project settings read/write
      BuildTools             Build, packages, console
      ExecuteTools           Run arbitrary C# in-editor
    Serialization/           GameObject, Asset, Property serializers
    UI/                      Status window (Window > Claude MCP)
    Utils/                   Undo helper, instance tracker
```

---

## Compatibility

| | Supported |
|---|---|
| **Unity** | 2021.3+ (tested on Unity 6.0) |
| **Render Pipeline** | URP, HDRP, Built-in |
| **OS** | macOS, Windows |
| **MCP Clients** | Claude Desktop, Claude Code, Cursor, or any MCP client |

---

## Contributing

Pull requests welcome! If you add a new tool, follow the pattern in `Editor/Tools/`. Update `SKILL.md` with usage examples and `WORKFLOW.md` if there are gotchas.

---

## License

[MIT](LICENSE) — use it however you want.
