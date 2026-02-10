# Agent Blender

An AI agent skill for Blender addon development and MCP-powered 3D scripting.

## Overview

**Agent Blender** is an expert skill that enables AI assistants to build Blender addons (plugins) using Python and the Blender Python API (`bpy`). It covers the full addon development lifecycle — from scaffolding and operator design to packaging and distribution.

## Contents

| File | Description |
|------|-------------|
| [`SKILL.md`](SKILL.md) | Core skill definition — addon patterns, API references, naming conventions, and anti-patterns |
| [`blender_mcp_guide.md`](blender_mcp_guide.md) | Beginner's guide to using Blender MCP for live AI-driven 3D scene control |

## Capabilities

- **Addon Scaffolding** — `bl_info` metadata, single-file and multi-file architectures
- **Operator Design** — Execute, invoke, modal, and poll patterns
- **Panel & UI Creation** — `bpy.types.Panel`, `UILayout` systems
- **Property Systems** — `bpy.props`, `PropertyGroup` for configurable values
- **Mesh Generation** — Procedural geometry with `bmesh` and `bpy.data.meshes`
- **Material Scripting** — PBR materials and shader node graphs via code
- **Keymap Registration** — Custom shortcuts and hotkey management
- **Addon Preferences** — Persistent global settings (`AddonPreferences`)
- **Packaging** — Legacy zip and Blender 4.2+ extension format (`blender_manifest.toml`)
- **MCP Integration** — Live Blender scripting through Model Context Protocol

## Usage

This skill is designed for use with AI coding assistants (like Antigravity). Load the `SKILL.md` as an agent skill to unlock expert-level Blender addon development guidance.

For live Blender control via AI, see the [Blender MCP Guide](blender_mcp_guide.md).

## Quick Start Keywords

Use these keywords to activate the skill:
`blender addon`, `blender plugin`, `blender scripting`, `bpy`, `blender python`, `blender operator`, `blender panel`, `blender extension`

## License

MIT
