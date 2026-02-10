# Blender MCP: Beginner's Guide

Blender MCP (Model Context Protocol) is a bridge that connects the power of Large Language Models (like Claude) directly to Blender. It allows you to use natural language to create, modify, and manage 3D scenes.

## What it does
- **Command Execution**: You can tell the AI to "Create a 3D city grid with 10 buildings" or "Rotate the selected object by 45 degrees."
- **Scripting Assistance**: It uses Blender's Python API under the hood, but you don't need to write any code yourself.
- **Agentic Workflow**: Unlike a simple chat assistant, it can plan multiple steps (e.g., "Add a cube, scale it, add a bevel modifier, and set the material to metallic") and execute them sequentially.

## How to use it

### 1. Enable the Sidebar
Once the addon is installed (see [Implementation Plan](file:///C:/Users/skdso/.gemini/antigravity/brain/e8f87233-00c7-4135-b68a-bb572042b317/implementation_plan.md) for installation steps), open Blender and:
- Press **`N`** in the 3D Viewport to open the sidebar.
- Click on the **`BlenderMCP`** tab.

### 2. Configure Antigravity (Me!)
Since I am your AI assistant, you don't need to install any external "brains" or API keys. I will talk to Blender for you.

To let me help you, just:
1. Make sure you have clicked **"Start MCP Server"** in the Blender sidebar (`BlenderMCP` tab).
2. Tell me what you want to do (e.g., "Create a 3D city").
3. I will use a small script to send commands directly to your Blender.

**No API keys or Cursor settings required!**

### 3. Connect
- In the `BlenderMCP` sidebar tab, click **Connect to Claude** (or the respective provider).
- Ensure the MCP server is running (usually handled automatically by the addon or your MCP host like Claude for Desktop).

### 4. Give Commands
You can now use your AI tool (Claude or Cursor) to control Blender. Try prompts like:
- "Create a procedurally generated forest of 50 trees."
- "Apply a red glass material to all spheres in the scene."
- "Organize my scene collection by object type."

## Troubleshooting
- **Connection Error**: Make sure you have an active internet connection and a valid API key.
- **Python Errors**: If a command fails, Blender MCP will report the error back to the AI, which will usually try to fix its own script and re-execute it.
