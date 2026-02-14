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

### API Compatibility and Versioning

**When to use**: Ensuring your addon works across Blender 3.6 LTS, 4.x, and future 5.x versions. Handle the transition from OpenGL to Vulkan (default in 5.0+).

```python
import bpy
import gpu

# Set version constants
BLENDER_40 = bpy.app.version >= (4, 0, 0)
BLENDER_45 = bpy.app.version >= (4, 5, 0) # Official Vulkan support
BLENDER_50 = bpy.app.version >= (5, 0, 0) # Vulkan by default

def get_backend_safe_shader(shader_type):
    """Handle built-in shader renames between OpenGL and Vulkan"""
    # Blender 4.5+ Vulkan backend often requires explicit shader types
    # Example: 'UNIFORM_COLOR' -> 'POINT_UNIFORM_COLOR' for points
    if BLENDER_45 and shader_type == 'UNIFORM_COLOR':
         # Use create_from_info for custom shaders or keep builtin logic branched
         pass 
    
    name = shader_type
    if BLENDER_40:
        name = name.replace("2D_", "").replace("3D_", "")
    return gpu.shader.from_builtin(name)

def set_gpu_state(line_width=1.0):
    """Vulkan/Metal don't support older global state line smoothing"""
    if not BLENDER_45:
        gpu.state.line_width_set(line_width)
    else:
        # 5.0+ prefers polyline shaders over global state
        # Logic for polyline shader setup would go here
        pass
```

**Common Version-Specific Changes:**
- **Blender 4.0**: Principled BSDF overhaul (renamed inputs), GPU shader names changed (removed `2D_`/`3D_` prefixes), Node socket storage changed.
- **Blender 4.2**: New Extensions system; prefers `blender_manifest.toml` over `bl_info` for distribution.
- **Blender 4.5 - 5.0 (Vulkan Transition)**: 
    - **Backends**: Official support for Vulkan (4.5) and making it default (5.0). 
    - **Shaders**: Prefer `gpu.shader.create_from_info` over raw GLSL strings. 4.5+ Built-in shaders are stricter (e.g., use `'POINT_UNIFORM_COLOR'` for point clouds).
    - **GPU State**: Global states like `line_width_set` and `point_size_set` are deprecated or behave differently in Vulkan; use thick line/polyline shaders instead.
    - **Attributes**: Use `gpu_extras.batch.batch_for_shader` to ensure vertex attributes match the backend's requirements.

### Master Level: Custom Matrix Gizmos

**When to use**: Creating intuitive 3D interfaces like custom transform tools, interactive handles, or on-screen controls that follow complex 4x4 matrix math.

```python
import bpy
from mathutils import Matrix

class MYADDON_GGT_custom_gizmos(bpy.types.GizmoGroup):
    bl_idname = "MYADDON_GGT_custom_gizmos"
    bl_label = "Custom Matrix Gizmos"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_options = {'3D', 'PERSISTENT'}

    @classmethod
    def poll(cls, context):
        return context.active_object is not None

    def setup(self, context):
        # Create a 3D arrow gizmo
        gz = self.gizmos.new("GIZMO_GT_arrow_3d")
        
        # Bind the gizmo to an object's property (e.g., location)
        gz.target_set_prop("offset", context.active_object, "location", index=0)
        
        gz.color = 0.1, 1.0, 0.5
        gz.alpha = 0.5
        gz.color_highlight = 1.0, 1.0, 1.0
        gz.alpha_highlight = 0.8
        
        self.handle_gizmo = gz

    def refresh(self, context):
        obj = context.active_object
        gz = self.handle_gizmo
        
        # Master Step: Update the gizmo's matrix_basis to match the object
        # This aligns the gizmo's local coordinate system with the object's world matrix
        gz.matrix_basis = obj.matrix_world.normalized()

def register():
    bpy.utils.register_class(MYADDON_GGT_custom_gizmos)

def unregister():
    bpy.utils.unregister_class(MYADDON_GGT_custom_gizmos)
```

**Key Matrix Gizmo Concepts:**
- **`matrix_basis`**: The 4x4 transformation matrix that defines the gizmo's position, rotation, and scale in the viewport.
- **`target_set_prop`**: Connects a gizmo's interactive action (like dragging) directly to a Python property or ID data.
- **`refresh()` vs `setup()`**: Use `setup` for creation and `refresh` for real-time matrix updates as the user moves objects or changes views.
- **Space Math**: Use `obj.matrix_world` for world-space gizmos, or construct custom matrices for tangent-space or screen-space handles.

### Modern Dependency Management (Blender 4.2+)

**When to use**: Including external Python libraries (e.g., NumPy, Requests) without manual installation.

```toml
# In blender_manifest.toml
[dependencies]
wheels = [
  "requests == 2.31.0",
  "numpy >= 1.26.0",
]

[permissions]
network = "Required to download textures from remote server"
files = "Required to export .log files to disk"
```

### Geometry Nodes & Node Group API (4.0+)

**When to use**: Procedural toolsets and custom modifiers. Note: 4.0+ uses the `node_tree.interface` for sockets.

```python
def create_geometry_node_group(name):
    # Create the tree
    group = bpy.data.node_groups.new(name, 'GeometryNodeTree')
    
    # Modern Socket Declaration (4.0+)
    # Old groups.inputs.new() is deprecated
    group.interface.new_socket(name="Size", in_out='INPUT', socket_type='NodeSocketFloat')
    group.interface.new_socket(name="Mesh", in_out='OUTPUT', socket_type='NodeSocketGeometry')
    
    nodes = group.nodes
    input_node = nodes.new('NodeGroupInput')
    output_node = nodes.new('NodeGroupOutput')
    
    # Create a simple Cube node
    cube = nodes.new('GeometryNodeMeshCube')
    group.links.new(input_node.outputs['Size'], cube.inputs['Size'])
    group.links.new(cube.outputs['Mesh'], output_node.inputs['Mesh'])
    
    return group
```

### App Handlers & Timers

**When to use**: Creating tools that react to the scene (e.g., auto-updating text) or running tasks in the background.

```python
@bpy.app.handlers.persistent
def on_frame_change(scene):
    print(f"Frame changed to: {scene.frame_current}")

def delayed_tool_execution():
    # Run once after 2 seconds
    print("Executing delayed task...")
    return None # Returning None stops the timer

def register():
    bpy.app.handlers.frame_change_post.append(on_frame_change)
    bpy.app.timers.register(delayed_tool_execution, first_interval=2.0)

def unregister():
    bpy.app.handlers.frame_change_post.remove(on_frame_change)
```

### Generic Attributes API (5.0 Ready)

**When to use**: Storing per-vertex or per-face data (UVs, Colors, Custom weights). Legacy methods like `vertex_colors` are deprecated.

```python
def add_custom_weight_attribute(obj):
    # Create a generic attribute on the Point (Vertex) domain
    attr = obj.data.attributes.new(
        name="CustomWeight",
        type='FLOAT',
        domain='POINT'
    )
    
    # Batch update values
    values = [0.5] * len(obj.data.vertices)
    attr.data.foreach_set("value", values)
```

### Cross-Platform Path Handling

**When to use**: Ensuring your addon works on Windows, macOS, and Linux. Never use backslashes `\` or hardcoded `/home/user` strings.

```python
import bpy
import os
from pathlib import Path

def get_addon_resources_path():
    """Get path to a folder inside your addon directory"""
    # Pathlib handles OS-specific separators automatically
    addon_dir = Path(__file__).parent
    resources_dir = addon_dir / "resources"
    return resources_dir

def get_blend_relative_path(filename):
    """Get an absolute path relative to the current .blend file"""
    if not bpy.data.is_saved:
        return None
    
    # Blender's '//' prefix means 'relative to blend file'
    # bpy.path.abspath converts it to a clean OS-native absolute path
    relative_path = f"//{filename}"
    return Path(bpy.path.abspath(relative_path))

def save_temp_data(data_name):
    """Use Blender's specific temp directory"""
    # Don't hardcode /tmp/ or C:\Temp\
    temp_dir = Path(bpy.app.tempdir)
    file_path = temp_dir / f"addon_{data_name}.txt"
    return file_path
```

**Cross-Platform Rules:**
- **Always use `pathlib.Path`**: It is the modern Python standard. Use the `/` operator to join paths.
- **Blender's `//` Prefix**: Always use `bpy.path.abspath("//...")` to resolve paths relative to the project file.
- **Escape Spaces**: If passing paths to external commands (via `subprocess`), always wrap paths in quotes or use `shlex.quote()`.
- **Case Sensitivity**: Remember that Linux/macOS are case-sensitive (`Texture.png` != `texture.png`), while Windows is not. Always use lowercase for asset filenames to avoid issues.

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
