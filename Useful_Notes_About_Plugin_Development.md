# Useful Notes about Plugin Development

## Basic loop of updating scene
```python
"""As found in bind_gaussian_splat_to_proxy_mesh.py"""
# For a given blender object
obj = bpy.data.objects.get(obj_name)

# Read from Depsgraph
depsgraph = bpy.context.evaluated_depsgraph_get()
eval_obj = obj.evaluated_get(depsgraph)
information = eval_obj.information # access specific information you might want

"""As found in sna_c2_refresh_all.py""
# Write to original
obj.data.vertices.foreach_set(property_name, updated_property)

# Mark mesh as dirty. Depsgraph re-evaluates automatically on the next access
obj.data.update() # Note this only updates obj and children
```


## Creating an empty and storing data in it
```python
"""As seen in load_from_blender_object"""
# Initialise the empty
empty_object = bpy.data.objects.new(object_name, None)

# Basic attributes of the empty (May not be necessary)
empty_object.empty_display_type = 'PLAIN_AXES'
empty_object.empty_display_size = 0.1
empty_object.matrix_world = source_obj.matrix_world.copy

# Store data in object properties
empty_object[property_name] = property

# Link to scene
bpy.context.collection.objects.link(empty_object)

# optional
# Initialise global cache if needed
if not hasattr(bpy, cache_name):
    bpy.cache_name = {}

# Add to global cache
bpy.cache_name[object_name] = {
    property_name : property
    ... # treat like a python dictionary
}
```


# TODO: Methods to update scene/objects periodically for render
# TODO: Methods to update scene/objects periodically for editing
# TODO: Caching of data and data access
