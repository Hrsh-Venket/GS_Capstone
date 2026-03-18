# Bugs
- Occurs when loading in lego_palace.ply. It also does this when I load duplicate.ply (an exported version of the same ply rotated to blender axes)
```
/home/hrsh/.config/blender/5.0/extensions/user_default/dgs_render_by_kiri_engine/src/sna_viewport_render.py:248: RuntimeWarning: invalid value encountered in cast
  depths_uint32 = depths_scaled.astype(np.uint32)
When I loaded lego_palace.ply -- nothing else



# Not bugs lol
- Understand what this is
```

Exception ignored in: <function ImagePreviewCollection.__del__ at 0x7f1fbf9f4a40>
Traceback (most recent call last):
  File "/home/hrsh/Software/blender-5.0.1-linux-x64/5.0/scripts/modules/bpy/utils/previews.py", line 63, in __del__
    raise ResourceWarning(
ResourceWarning: <ImagePreviewCollection id=0x7f1fb4f4e750[8], <super: <class 'ImagePreviewCollection'>, <ImagePreviewCollection object>>>: left open, remove with 'bpy.utils.previews.remove()'
Exception ignored in: <function ImagePreviewCollection.__del__ at 0x7f1fbf9f4a40>
Traceback (most recent call last):
  File "/home/hrsh/Software/blender-5.0.1-linux-x64/5.0/scripts/modules/bpy/utils/previews.py", line 63, in __del__
    raise ResourceWarning(
ResourceWarning: <ImagePreviewCollection id=0x7f1fb83ea8d0[8], <super: <class 'ImagePreviewCollection'>, <ImagePreviewCollection object>>>: left open, remove with 'bpy.utils.previews.remove()'
/home/hrsh/.config/blender/5.0/extensions/user_default/dgs_render_by_kiri_engine/src/sna_viewport_render.py:248: RuntimeWarning: invalid value encountered in cast
  depths_uint32 = depths_scaled.astype(np.uint32)

```
