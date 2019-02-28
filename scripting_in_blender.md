---
layout: default
title: Scripting in Blender-2.8
nav_order: 2
---

# Scripting in Blender-2.8
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Getting started

Open a new, default Blender document. Change the Timeline window to a Python Console. You might want to increase the height so you can see more lines at once.

Start out by typing:

```python
list(bpy.data.objects)
```

You should see the three default objects listed:

```python
[bpy.data.objects['Camera'], bpy.data.objects['Cube'], bpy.data.objects['Light']]
```

Now, let's reference the default cube for convenience:

```python
cube = bpy.data.objects["Cube"]
```

Next, we're going to move the cube, but first we need to import the ```mathutils``` module:

```python
import mathutils
```

Now we can move the default cube:

```python
cube.delta_location += mathutils.Vector((1, 1, 1))
```

Keep moving the cube along this trajectory by pressing the [UP] arrow and then pressing [ENTER]. Repeat.

Congratulations, you've just started scripting in Blender!

## The `bpy` module

The `bpy` module consists of several submodules:

| Submodule      | Description                                               |
|:---------------|:----------------------------------------------------------|
| `bpy.data`     | contents of the current document.                         |
| `bpy.types`    | info on types of objects in `bpy.data`.                   |
| `bpy.ops`      | _operations_ perform the actual functions of Blender.     |
| `bpy.context`  | settings such as currect 3D mode, selected objects, etc.  |
| `bpy.props`    | functions for defining _properties_.                      |

## The `mathutils` module

The `mathutils` module contains many usesful classes:

| Class         | Description                                                |
|:--------------|:-----------------------------------------------------------|
| `Vector`      | representation of 2D or 3D coordinates.                    |
| `Matrix`      | general way of representing a linear transformation.       |
| `Euler`       | way of representing rotations as a set of _Euler angles_.  |
| `Quaternion`  | another way of representing rotations.                     |
| `Color`       | representation of RGB colors and conversion to/from HSV.   |

## Adding a UV sphere to a scene

Clicking the _Scripting_ tab opens a text editor where we can write and execute code. Try copying and pasting the following script:

```python
import bpy
import bmesh

# Delete all existing objects
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# Create an empty mesh and the object
mesh = bpy.data.meshes.new("Ball")
ball = bpy.data.objects.new("Ball", mesh)

# Add object to scene and select it
bpy.context.collection.objects.link(ball)
ball.select_set(state=True)

# Construct the UV sphere and assign it to the Blender mesh
bm = bmesh.new()
bmesh.ops.create_uvsphere(bm, u_segments=24, v_segments=24, diameter=3)
bm.to_mesh(mesh)
bm.free()
```

Then click _Run Script_. You'll see the three default objects disappear, replaced by a UV sphere called "Ball".

With the object still selected, we can reposition the Ball object as follows:

```python
bpy.ops.transform.translate(value=(x, y, z))
```

Where `x`, `y`, and `z` correspond to the coordinates in Euclidean space. If we want to use smooth shading (again with the object still selected) we can add:

```python
bpy.ops.object.shade_smooth()
```

Let's modify our script so that we create 50 balls randomly positioned within a 25 by 25 by 25 unit cubic space:

```python
from random import randint

import bpy
import bmesh

def add_ball(x, y, z):
    mesh = bpy.data.meshes.new("Ball")
    ball = bpy.data.objects.new("Ball", mesh)
    bpy.context.collection.objects.link(ball)
    ball.select_set(state=True)
    bm = bmesh.new()
    bmesh.ops.create_uvsphere(bm, u_segments=12, v_segments=12, diameter=1)
    bm.to_mesh(mesh)
    bm.free()
    # Reposition the ball
    bpy.ops.transform.translate(value=(x, y, z))
    # Apply smooth shading
    bpy.ops.object.shade_smooth()
    # Deselect the ball
    ball.select_set(state=False)

# Clear the scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# Add 50 balls at random positions within a 25 by 25 by 25 unit cubic space
for i in range(50):
    add_ball(randint(1, 25), randint(1, 25), randint(1, 25))
```

## Creating materials and assigning them to meshes

To create a new material, called _Blue_, use the following syntax:

```python
bpy.data.materials.new("Blue")
```

If we want to make _Blue_ reflect its name:

```python
bpy.data.materials["Blue"].diffuse_color = (0, 0, 1, 1)
```

Say we have an object, _obj_, and we want to apply to it material _Blue_:

```python
obj.data.materials.append(bpy.data.materials["Blue"])
```

Let's try this in an example! The script below creates three cubes: _Cube1_, _Cube2_, and _Cube3_, each with their own materials: _Color1_, _Color2_, _Color3_, respectively:

```python
from random import random

import bpy
import bmesh

def create_and_apply_material(obj, tag):
    material_name = "Color{}".format(tag)
    m = bpy.data.materials.new(material_name)
    m.diffuse_color = (random(), random(), random(), 1)
    obj.data.materials.append(bpy.data.materials[material_name])

def add_cube(x, y, z, tag):
    mesh_name = "Cube{}".format(tag)
    mesh = bpy.data.meshes.new(mesh_name)
    cube = bpy.data.objects.new(mesh_name, mesh)
    bpy.context.collection.objects.link(cube)
    cube.select_set(state=True)
    bm = bmesh.new()
    bmesh.ops.create_cube(bm)
    bm.to_mesh(mesh)
    bm.free()
    
    # Resize the mesh
    bpy.ops.transform.resize(value=(2, 2, 2))
    
    # Reposition the cube
    bpy.ops.transform.translate(value=(x, y, z))
    
    # Call create_and_apply_material
    create_and_apply_material(cube, tag)    

    # Deselect the ball
    cube.select_set(state=False)

# Clear the scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# Add 3 cubes with randomly-colored materials
for i in range(3):
    add_cube(i * 2, i * 2, i * 2, i)
```

## Using modifiers and rendering a scene

Here's an example where we add a monkey to an empty scence and apply a few modifiers. Then we add a camera, point it at the monkey, and render the scene.

To apply the modifiers, after we add the monkey mesh to the scene:

```python
bpy.ops.object.modifier_add(type='SUBSURF')
bpy.ops.object.modifier_add(type='WIREFRAME')
```

This will apply a subsurface and wireframe modifier to the monkey mesh.

Next, we need to add a light source:

```python
bpy.ops.object.light_add(type='POINT', location=(2, 2, 2))
```

Then a camera:

```python
bpy.ops.object.camera_add(location=(-4, -5, 2))
bpy.context.scene.camera = bpy.data.objects["Camera"]
```

Then we'll add a constraint to the camera so that it points at the monkey (recall that by default Blender names monkeys `Suzanne`):

```python
bpy.ops.object.constraint_add(type="TRACK_TO")
bpy.context.object.constraints["Track To"].target = bpy.data.objects["Suzanne"]
bpy.context.object.constraints["Track To"].track_axis = "TRACK_NEGATIVE_Z"
bpy.context.object.constraints["Track To"].up_axis = "UP_Y"
```

Finally, we can render the scene:

```python
bpy.data.scenes["Scene"].render.filepath = "render.png"
bpy.ops.render.render(write_still=True)
```

Putting it all together:

```python
import bpy
import bmesh

# Clear the scene
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# Add a monkey
bpy.ops.mesh.primitive_monkey_add()

# Apply some modifiers
bpy.ops.object.modifier_add(type='SUBSURF')
bpy.ops.object.modifier_add(type='WIREFRAME')

# Add a light source
bpy.ops.object.light_add(type='POINT', location=(2, 2, 2))

# Change background to black
bpy.context.scene.world.use_nodes = False
bpy.context.scene.world.color = (0, 0, 0)

# Add a camera
bpy.ops.object.camera_add(location=(-4, -5, 2))
bpy.context.scene.camera = bpy.data.objects["Camera"]

# Point camera at the monkey
bpy.ops.object.constraint_add(type="TRACK_TO")
bpy.context.object.constraints["Track To"].target = bpy.data.objects["Suzanne"]
bpy.context.object.constraints["Track To"].track_axis = "TRACK_NEGATIVE_Z"
bpy.context.object.constraints["Track To"].up_axis = "UP_Y"

# Render scene
bpy.data.scenes["Scene"].render.filepath = "render.png"
bpy.ops.render.render(write_still=True)
```

## Further reading on Python API for Blender

To learn more about the Python API for Blender, please see the [this link](https://docs.blender.org/api/current/). Note: this hasn't been updated for Blender-2.8 so if you have specific questions, feel free to e-mail me at `a.r.kaija@gmail.com` and I'll try to help!

## Building an _addon_

_Addons_ are Python scripts that extend Blender's functionality. They can reside in two places:

1. A file within the Blender user preferences directory: in this case the addon needs to be _enabled_ in each Blender document where it is to be used, by ticking its checkbox in the Add-Ons list in the User Preferences window.

2. A text block within the Blender document: in this case the script can be run by clicking _Run Script_ in the Text Editor window, or by ticking the _Register_ checkbox in the Text Editor window.

Typically, an addon script defines one or more new _operators_, as a subclass of the `bpy.types.Operator` class. Your class must be given a unique name within the document. An operator must have a `bl_idname` attribute, giving the operator a name, and a `bl_label`, giving it a user-visible name in the F3 menu. The `bl_idname` has the following format, for example for the built-in operator to add a cube:

```python
mesh.primitive_cube_add
```

Where the part to the left of the single dot is a valid name for a category of operators, which can be found by entering the following in the Python console:

```python
dir(bpy.ops)
```

Now we'll demonstrate how to define an operator that adds a new tetrahedron object to the scene. Our class definition might look something like this:

```python
class AddTetrahedron(bpy.types.Operator):
    bl_idname = "mesh.add_tetrahedron"
    bl_label = "Add Tetrahedron"
```

The class must define an `invoke` method like this:

```python
def invoke(self, context, event):
```

Where the `invoke` method carries out the actual function of the operator. After it finishes, it must return a set of strings, telling Blender that the operation has finished. We'll end our `invoke` method like this:

```python
return {"FINISHED"}
```

We won't get into explaining the details about how the geometry works, but first we use the `mathutils` module to define the vertices of our mesh:

```python
vertices = [Vector((0, -1 / sqrt(3), 0)),
            Vector((0.5, 1 / (2 * sqrt(3)), 0)),
            Vector((-0.5, 1 / (2 * sqrt(3)), 0)),
            Vector((0, 0, sqrt(2 / 3))),]
```

Then we create a mesh:

```python
new_mesh = bpy.data.meshes.new("Tetrahedron")
```

Then fill the mesh with the vertices definitions and resulting faces:

```
new_mesh.from_pydata(vertices, [], [[0, 1, 2], [0, 1, 3], [1, 2, 3], [2, 0, 3]])
```

Where `mesh.from_pydata` takes three arguments:

1. A list of `Vector`s defining the vertices.

2. A list of edge definitions.

3. A list of face definitions.

Edges or faces are defined as lists of vertex indices - in this example we have four faces, each defined by three vertices. __You can either pass the edge definitions or the face definitions, but not both.__ This is why we've passed an empty list for the edge definitions.

Since we've changed the mesh we need to update it:

```python
new_mesh.update
```

And create and object and link it to the scene as we did in our previous examples. Putting everything together:

```python
from math import sqrt
from mathutils import Vector

import bpy

class AddTetrahedron(bpy.types.Operator):
    bl_idname = "mesh.add_tetrahedron"
    bl_label = "Add Tetrahedron"

    def invoke(self, context, event):
        vertices = [Vector((0, -1 / sqrt(3), 0)),
                    Vector((0.5, 1 / (2 * sqrt(3)), 0)),
                    Vector((-0.5, 1 / (2 * sqrt(3)), 0)),
                    Vector((0, 0, sqrt(2 / 3))),]
        new_mesh = bpy.data.meshes.new("Tetrahedron")
        new_mesh.from_pydata(vertices, [], [[0, 1, 2], [0, 1, 3], [1, 2, 3], [2, 0, 3]])
        new_mesh.update()
        new_object = bpy.data.objects.new("Tetrahedron", new_mesh)
        bpy.context.collection.objects.link(new_object)
        return {"FINISHED"}

bpy.utils.register_class(AddTetrahedron)
```

The call to `register_class` adds the operator to Blender's built-in collection. Try copying the code above in to the Blender text editor and clicking `Run Script`. Then press F3 to search for the addon, typing "tetra" - the _Add Tetrahedron_ addon should appear, congratulations!

## Making addons installable

To make an addon installable, first we need to define a global called `bl_info`, a Python dictionary. Make sure to enter `"blender" : (2, 80, 0)`! Here's an example:

```python
bl_info = {"name" : "Add Tetrahedron",
           "author" : "John Doe <john.doe@example.com>",
           "location" : "View3D > Add > Mesh > Generate",
           "version" : (1, 0, 0),
           "blender" : (2, 80, 0),
           "description" : "Add a tetrahedron mesh",
           "category" : "Add Mesh",}
```

Then, we need to reformat our call to `bpy.utils.register_class` as follows:

```python
classes = (AddTetrahedron,)

def register():
    for cls in classes:
        bpy.utils.register_class(cls)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
```

Putting it all together:

```python
bl_info = {"name" : "Add Tetrahedron",
           "author" : "John Doe <john.doe@example.com>",
           "location" : "View3D > Add > Mesh > Generate",
           "version" : (1, 0, 0),
           "blender" : (2, 80, 0),
           "description" : "Add a tetrahedron mesh",
           "category" : "Add Mesh",}

from math import sqrt
from mathutils import Vector

import bpy

class AddTetrahedron(bpy.types.Operator):
    bl_idname = "mesh.add_tetrahedron"
    bl_label = "Add Tetrahedron"

    def invoke(self, context, event):
        vertices = [Vector((0, -1 / sqrt(3), 0)),
                    Vector((0.5, 1 / (2 * sqrt(3)), 0)),
                    Vector((-0.5, 1 / (2 * sqrt(3)), 0)),
                    Vector((0, 0, sqrt(2 / 3))),]
        new_mesh = bpy.data.meshes.new("Tetrahedron")
        new_mesh.from_pydata(vertices, [], [[0, 1, 2], [0, 1, 3], [1, 2, 3], [2, 0, 3]])
        new_mesh.update()
        new_object = bpy.data.objects.new("Tetrahedron", new_mesh)
        bpy.context.collection.objects.link(new_object)
        return {"FINISHED"}

classes = (AddTetrahedron,)

def register():
    for cls in classes:
        bpy.utils.register_class(cls)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
```

Try saving the code above into a `.py` file, then install the addon:

1. Open Blender.

2. Click _Edit_ > _Preferences_.

3. Click the _Add-ons_ tab.

4. Click _Install..._ and then select the file you saved earlier.

5. Enable the addon by ticking the box next to its name, for this example: _Add Mesh: Add Tetrahedron_.

And that's it! Congratulations, you're now ready to start building your own addons for Blender-2.8!
