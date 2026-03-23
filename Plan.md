# Software plan

This document outlines the plan of how I will implement rigging for Gaussian Splats as an addon for Blender. 

Aim of the software: Complement existing workflows for artists who do rigging on blender, and work (or want to work with 3DGS). Thus:


## Structure

- IO with ply file:
    - Uses existing blender IO operations to load gaussian splat ply as a mesh ply with special attributes
    - Uses special attributes to convert this mesh to a gaussian splat
    - [ ] Add logging to this convertion and IO to check if a user does not upload a valid ply file
- Handles Guassians
    - Edit
    - Render
    - Interestingly uses dbscan to auto crop objects. Can use this to more easily paint gaussians onto a rig
- Modifier operations
    - Crop Box
    - Colour Edit
    - Keyframe-based animation operationsA


## Idea for rigging implementation (cpu prototype)
- Gaussian is loaded as a source mesh with special properties
    - Each gaussian is a vertex with attributes of a gaussian
    - Therefore, each vertex can be rigged onto the bones natively in blender
- We need to adjust the viewport renderer from blender to translate and rotate gaussians rigged to specific bones
    - Each gaussian (vertex) can be connected to n bones
    - Get R and t from bpy.PoseBone.matrix which returns a 4*4 array (Source: https://blenderartists.org/t/whats-the-difference-of-pose-bones-matrix-basis-matrix-matrix-channel/529837/2)
    - Translation can be a weighted average of the position of the gaussian (based on the pull of all bones)
    - Rotation can be taken as the weighted average of the n quaternions
        - weighted average method: https://stackoverflow.com/questions/12374087/average-of-multiple-quaternions


- Rotation is less important. First iteration only translation or use easy rotation interpretation
- 

## TODO
- [x] Restructure code, understand it in detail and mark TODOs
- [ ] Add operations for rigging (discuss operations)
    - [ ] Translation of gaussians based on weighted average of rigged bone positions
    - [ ] 
 - [ ] Rewrite math operations, try to vectorise or similar
 - [ ] Add proper logging (only print debugging exists across this repository)
- [ ] Test operations
- [ ] Create basic GUI for operations
- [ ] Improve GUI based on feedback
- [ ] Structure IO for rigged (FBX? GLTF?)
- [ ] Look into optimisations given freethreading in python 3.14

# Checkpoints to build addon

- March 13: 
- March 20: Restructure code, understand it in detail and mark TODOs
- March 27: 
- April 10: Write basic operations and add a gui
- April 17:
- April 24:
- May 1:
- May 8:
- May 15:
- May 22:
- May 29: Interim Report Submission

