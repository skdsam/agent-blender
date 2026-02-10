---
name: blender-addon-development
description: "Expert in building Blender addons (plugins) using Python and the Blender Python API (bpy). Covers addon architecture, operator design, panel/UI creation, property systems, mesh generation, material scripting, keymap registration, preferences, and packaging for distribution. Use when: blender addon, blender plugin, blender scripting, bpy, blender python, blender operator, blender panel, blender extension."
---

# Blender Addon Development

**Role**: Blender Addon Architect

You build addons that extend Blender's power. You understand the bpy API,
Blender's data model, operator lifecycle, and UI patterns. You create tools
that feel native to Blender and follow its conventions. You know when to use
operators vs. modal operators, how to manage undo correctly, and how to
package addons for the community.

## Capabilities

- Addon scaffolding and `bl_info` metadata
- Operator design (execute, invoke, modal, poll)
- Panel and UI layout (`bpy.types.Panel`, `UILayout`)
- Property systems (`bpy.props`, `PropertyGroup`)
- Mesh creation and manipulation (`bmesh`, `bpy.data.meshes`)
- Material and shader node scripting
- Keymap and shortcut registration
- Addon preferences (`AddonPreferences`)
- Multi-file addon architecture
- Packaging and distribution (legacy zip and Blender 4.2+ extensions)
- MCP server integration for live Blender scripting

## Patterns

### Single-File Addon Scaffold

Minimal addon with an operator and a panel.

**When to use**: Quick tools, simple one-operator addons

```python
bl_info = {
    "name": "My Addon",
    "author": "Your Name",
    "version": (1, 0, 0),
    "blender": (4, 0, 0),
    "location": "View3D > Sidebar > My Addon",
    "description": "Short description of what it does",
    "category": "Object",
}

import bpy

class MYADDON_OT_do_thing(bpy.types.Operator):
    """Tooltip for the operator"""
    bl_idname = "myaddon.do_thing"
    bl_label = "Do Thing"
    bl_options = {'REGISTER', 'UNDO'}

    @classmethod
    def poll(cls, context):
        return context.active_object is not None

    def execute(self, context):
        obj = context.active_object
        obj.location.x += 1.0
        self.report({'INFO'}, "Done!")
        return {'FINISHED'}

class MYADDON_PT_main_panel(bpy.types.Panel):
    bl_label = "My Addon"
    bl_idname = "MYADDON_PT_main_panel"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "My Addon"

    def draw(self, context):
        layout = self.layout
        layout.operator("myaddon.do_thing")

classes = (
    MYADDON_OT_do_thing,
    MYADDON_PT_main_panel,
)

def register():
    for cls in classes:
        bpy.utils.register_class(cls)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)

if __name__ == "__main__":
    register()
```

### Multi-File Addon Structure

**When to use**: Complex addons with many operators, panels, or sub-modules

```
my_addon/
├── __init__.py          # bl_info, register/unregister
├── operators/
│   ├── __init__.py
│   ├── mesh_ops.py      # Mesh-related operators
│   └── material_ops.py  # Material operators
├── panels/
│   ├── __init__.py
│   └── main_panel.py    # UI panels
├── properties.py        # PropertyGroup definitions
├── preferences.py       # AddonPreferences
├── utils.py             # Shared helper functions
└── icons/               # Custom icons (optional)
    └── custom_icon.png
```

**`__init__.py` pattern for multi-file addons:**

```python
bl_info = {
    "name": "My Complex Addon",
    "author": "Your Name",
    "version": (1, 0, 0),
    "blender": (4, 0, 0),
    "location": "View3D > Sidebar > My Addon",
    "description": "A complex multi-file addon",
    "category": "Object",
}

import bpy
from . import operators, panels, properties, preferences

modules = (properties, preferences, operators, panels)

def register():
    for mod in modules:
        mod.register()

def unregister():
    for mod in reversed(modules):
        mod.unregister()
```

### Property System

**When to use**: Storing user-configurable values, per-object data

```python
import bpy
from bpy.props import (
    StringProperty, IntProperty, FloatProperty,
    BoolProperty, EnumProperty, FloatVectorProperty,
    PointerProperty,
)

class MYADDON_PG_settings(bpy.types.PropertyGroup):
    my_string: StringProperty(name="Name", default="Hello")
    my_int: IntProperty(name="Count", default=5, min=1, max=100)
    my_float: FloatProperty(name="Scale", default=1.0, min=0.01, max=10.0)
    my_bool: BoolProperty(name="Enable", default=True)
    my_enum: EnumProperty(
        name="Mode",
        items=[
            ('OPT_A', "Option A", "First option"),
            ('OPT_B', "Option B", "Second option"),
        ],
        default='OPT_A',
    )
    my_color: FloatVectorProperty(
        name="Color", subtype='COLOR',
        default=(1.0, 0.5, 0.0), min=0.0, max=1.0,
    )

def register():
    bpy.utils.register_class(MYADDON_PG_settings)
    bpy.types.Scene.my_addon = PointerProperty(type=MYADDON_PG_settings)

def unregister():
    del bpy.types.Scene.my_addon
    bpy.utils.unregister_class(MYADDON_PG_settings)
```

**Accessing in operators/panels:**
```python
settings = context.scene.my_addon
print(settings.my_string)
```

### Mesh Generation with bmesh

**When to use**: Procedural geometry, custom mesh tools

```python
import bpy
import bmesh

class MYADDON_OT_create_grid(bpy.types.Operator):
    bl_idname = "myaddon.create_grid"
    bl_label = "Create Custom Grid"
    bl_options = {'REGISTER', 'UNDO'}

    size: bpy.props.FloatProperty(name="Size", default=2.0)
    subdivisions: bpy.props.IntProperty(name="Subdivisions", default=4, min=1)

    def execute(self, context):
        mesh = bpy.data.meshes.new("CustomGrid")
        obj = bpy.data.objects.new("CustomGrid", mesh)
        context.collection.objects.link(obj)
        context.view_layer.objects.active = obj

        bm = bmesh.new()
        bmesh.ops.create_grid(
            bm, x_segments=self.subdivisions,
            y_segments=self.subdivisions, size=self.size
        )
        bm.to_mesh(mesh)
        bm.free()

        return {'FINISHED'}
```

### Material and Shader Node Scripting

**When to use**: Programmatic material creation, batch material editing

```python
import bpy

def create_pbr_material(name, base_color=(0.8, 0.2, 0.1, 1.0), roughness=0.5):
    mat = bpy.data.materials.new(name=name)
    mat.use_nodes = True
    nodes = mat.node_tree.nodes
    links = mat.node_tree.links

    # Clear defaults
    nodes.clear()

    # Create nodes
    output = nodes.new('ShaderNodeOutputMaterial')
    bsdf = nodes.new('ShaderNodeBsdfPrincipled')

    # Set values
    bsdf.inputs['Base Color'].default_value = base_color
    bsdf.inputs['Roughness'].default_value = roughness

    # Link
    links.new(bsdf.outputs['BSDF'], output.inputs['Surface'])

    # Position nodes for readability
    output.location = (300, 0)
    bsdf.location = (0, 0)

    return mat
```

### Modal Operator

**When to use**: Interactive tools (mouse-driven, real-time feedback, gizmos)

```python
class MYADDON_OT_modal_draw(bpy.types.Operator):
    bl_idname = "myaddon.modal_draw"
    bl_label = "Modal Draw"

    def modal(self, context, event):
        context.area.tag_redraw()

        if event.type == 'MOUSEMOVE':
            # React to mouse movement
            self.mouse_x = event.mouse_region_x
            self.mouse_y = event.mouse_region_y
            return {'RUNNING_MODAL'}
        elif event.type == 'LEFTMOUSE' and event.value == 'PRESS':
            # Confirm action
            return {'FINISHED'}
        elif event.type in {'RIGHTMOUSE', 'ESC'}:
            # Cancel
            return {'CANCELLED'}

        return {'RUNNING_MODAL'}

    def invoke(self, context, event):
        self.mouse_x = 0
        self.mouse_y = 0
        context.window_manager.modal_handler_add(self)
        return {'RUNNING_MODAL'}
```

### Addon Preferences

**When to use**: Global settings that persist across sessions

```python
class MYADDON_AP_preferences(bpy.types.AddonPreferences):
    bl_idname = __package__  # Must match addon module name

    api_key: bpy.props.StringProperty(
        name="API Key", subtype='PASSWORD',
    )
    default_path: bpy.props.StringProperty(
        name="Default Export Path", subtype='DIR_PATH',
    )

    def draw(self, context):
        layout = self.layout
        layout.prop(self, "api_key")
        layout.prop(self, "default_path")
```

**Accessing preferences:**
```python
prefs = bpy.context.preferences.addons[__package__].preferences
print(prefs.api_key)
```

### Blender 4.2+ Extension Format (blender_manifest.toml)

**When to use**: Distributing addons for Blender 4.2+ via Extensions platform

```toml
schema_version = "1.0.0"

id = "my_addon"
version = "1.0.0"
name = "My Addon"
tagline = "Short one-line description"
maintainer = "Your Name <email@example.com>"
type = "add-on"

# Compatible Blender versions
blender_version_min = "4.2.0"

# License (SPDX identifier)
license = ["SPDX:GPL-3.0-or-later"]

# Optional
website = "https://github.com/yourname/my-addon"
tags = ["Object", "Mesh"]

# Python dependencies (pip packages)
# [permissions]
# network = "For update checking"
```

**Building extension package:**
```bash
# From the addon directory
blender --command extension build --source-dir ./my_addon --output-dir ./dist
```

### Naming Conventions

Blender enforces strict naming for classes:

| Type | Pattern | Example |
|------|---------|---------|
| Operator | `ADDONNAME_OT_action_name` | `MYADDON_OT_create_grid` |
| Panel | `ADDONNAME_PT_panel_name` | `MYADDON_PT_main_panel` |
| Menu | `ADDONNAME_MT_menu_name` | `MYADDON_MT_context_menu` |
| PropertyGroup | `ADDONNAME_PG_group_name` | `MYADDON_PG_settings` |
| UIList | `ADDONNAME_UL_list_name` | `MYADDON_UL_item_list` |
| Header | `ADDONNAME_HT_header_name` | `MYADDON_HT_main_header` |
| Gizmo | `ADDONNAME_GZ_gizmo_name` | `MYADDON_GZ_transform` |
| GizmoGroup | `ADDONNAME_GGT_group_name` | `MYADDON_GGT_transform` |
| AddonPreferences | `ADDONNAME_AP_preferences` | `MYADDON_AP_preferences` |

### Keymap Registration

```python
addon_keymaps = []

def register_keymaps():
    wm = bpy.context.window_manager
    kc = wm.keyconfigs.addon
    if kc:
        km = kc.keymaps.new(name='3D View', space_type='VIEW_3D')
        kmi = km.keymap_items.new(
            "myaddon.do_thing", type='D', value='PRESS',
            ctrl=True, shift=True
        )
        addon_keymaps.append((km, kmi))

def unregister_keymaps():
    for km, kmi in addon_keymaps:
        km.keymap_items.remove(kmi)
    addon_keymaps.clear()
```

## Anti-Patterns

### ❌ Not Using `bl_options = {'REGISTER', 'UNDO'}`

**Why bad**: Users can't undo the operator's changes.
Breaks Blender's undo stack expectations.

**Instead**: Always include `'REGISTER', 'UNDO'` for operators that modify data.
Only omit for query-only operators.

### ❌ Hardcoding Paths

**Why bad**: Breaks across OS (Windows vs Linux vs Mac).
Fails when Blender runs portably.

**Instead**: Use `bpy.path.abspath("//")` for blend-relative paths.
Use `bpy.app.tempdir` for temp files.
Use `AddonPreferences` for user-configurable paths.

### ❌ Operating Outside Context

**Why bad**: `context.active_object` may be None.
Area type may not match expectations.
Causes crashes or cryptic errors.

**Instead**: Always implement `poll()` to check context.
Guard against None objects with early returns.
Use `context.temp_override()` when needed (Blender 3.2+).

### ❌ Not Cleaning Up on Unregister

**Why bad**: Leaves orphan data in Blender.
Causes errors after addon disable.
Memory leaks persist.

**Instead**: Delete custom properties (`del bpy.types.Scene.my_addon`).
Remove keymaps.
Unregister classes in reverse order.
Clean up handlers (`bpy.app.handlers`).

### ❌ Using Global State Instead of Properties

**Why bad**: State lost between undo/redo.
Not serialized with .blend file.
Threading issues in background execution.

**Instead**: Use `PropertyGroup` attached to Scene/Object.
Use operator properties for operator-specific state.
Use `AddonPreferences` for persistent settings.

## Useful API References

| Task | Module / Function |
|------|-------------------|
| Scene objects | `bpy.context.scene.objects` |
| Active object | `bpy.context.active_object` |
| Selected objects | `bpy.context.selected_objects` |
| Create mesh | `bpy.data.meshes.new()` |
| Create object | `bpy.data.objects.new()` |
| Link to collection | `bpy.context.collection.objects.link(obj)` |
| Edit mode mesh | `bmesh.from_edit_mesh(obj.data)` |
| Run operator | `bpy.ops.mesh.primitive_cube_add()` |
| Timer/handler | `bpy.app.timers.register()` |
| File handler | `bpy.app.handlers.load_post` |
| User report | `self.report({'INFO'}, "message")` |

## Related Skills

Works well with: `3d-web-experience`, `python-patterns`, `game-development`
