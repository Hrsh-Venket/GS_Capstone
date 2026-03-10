# Software plan

This document outlines the plan of how I will implement rigging for Gaussian Splats as an addon for Blender. 

Aim of the software: Complement existing workflows for artists who do rigging on blender, and work (or want to work with 3DGS). Thus:


## Structure
- IO with PLY file: plyfile.py
    - Can use existing code here
    - Validate input and have verbose errors (need to add)
- UI elements with Serpens blender addon: __init.py__
- Handles Gaussians: __init.py__
    - Edit (manipulate gaussians)
    - Render (some GPU code here)
    - Mesh 2 3DGS (need to understand which algorithms they use)
- Modifier operations: __init.py__
    - Crop Box, Colour Edit, Keyframe-based animation operations, etc

## TODO
- [ ] Restructure code, understand it in detail and mark TODOs
- [ ] Add operations for rigging (discuss operations)
    - [ ] Paint gaussians onto the bones
    - [ ] link gaussians onto the bones
    - [ ] 
- [ ] Test operations
- [ ] Create basic GUI for operations
- [ ] Improve GUI based on feedback
- [ ] Structure IO for rigged (FBX? GLTF?)

# Checkpoints to build addon

- March 13: Restructure code, understand it in detail and mark TODOs
- March 20: 
- March 27: Write basic operations guiv
- April 10:
- April 17:
- April 24:
- May 1:
- May 8:
- May 15:
- May 22:
- May 29: Interim Report Submission

