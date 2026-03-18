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
    - Keyframe-based animation operations

## TODO
- [ ] Restructure code, understand it in detail and mark TODOs
 - [ ] Rewrite math operations, try to vectorise or similar
 - [ ] Add proper logging (only print debugging exists across this repository)
- [ ] Add operations for rigging (discuss operations)
    - [ ] Paint gaussians onto the bones
    - [ ] Link gaussians onto the bones
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

