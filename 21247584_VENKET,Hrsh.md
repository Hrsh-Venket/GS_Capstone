# Interim Report: Blender Addon for Rigging 3D Gaussian Splats

Author: VENKET, Hrsh
Student ID: 21247584 


## TODO:
- [ ] Achievement
- [ ] Evidence
- [X] Minutes of >= Meetings with supervisors


# Interim Report: Blender Addon for Rigging 3D Gaussian Splats

## Achievement

### Proxy-Mesh Binding with Topology-Aware Assignment

The core operation of the addon is binding a 3D Gaussian Splat to a proxy mesh such that deformations applied to the mesh transfer naturally to the splat. The fundamental unit of this binding is the assignment of each gaussian to a mesh face (triangle). Once bound, any deformation of that triangle is propagated to its assigned gaussians.

A naive approach to this assignment is to bind each gaussian to the triangle whose surface is closest to the gaussian's centre. This is straightforward to compute using barycentric coordinates — effectively an orthogonal projection of each gaussian centre onto its nearest triangle. However, this approach fails in geometrically ambiguous regions. For example, in the case of a hand, gaussians representing one finger may be geometrically closer to mesh faces belonging to an adjacent finger, resulting in incorrect assignments that produce severe artifacts during deformation.

To address this, a priority-queue-based assignment scheme was implemented. The queue ranks candidate assignments using two metrics: first, the difference in distance between the closest and second-closest triangles (a measure of assignment ambiguity), and second, a penalty based on the topological distance on the mesh from gaussians that have already been assigned. On the first iteration, only the distance-difference metric is used. In subsequent iterations, the topological penalty is introduced and the queue is updated accordingly. This encourages spatially coherent assignments that respect the mesh's surface topology rather than relying purely on Euclidean proximity.

### TBN-Space Deformation Transfer

Once a gaussian is bound to a triangle, deformations are transferred from the mesh to the gaussian across three channels.

For position, the gaussian's location is stored in TBN (tangent, bitangent, normal) coordinates relative to its bound triangle. When the triangle deforms, the gaussian's position is recomputed from these stored TBN coordinates, ensuring it moves consistently with the triangle surface.

For rotation, a rotation delta relative to the gaussian's original orientation at bind time is maintained. Using the original delta rather than an incremental update ensures idempotency — the result is the same regardless of how many intermediate update checks have occurred.

For scaling, TBN ratios between the gaussian's dimensions and the triangle's dimensions are stored at bind time. On deformation, these ratios are used to apply an affine transform to the gaussian, scaling it proportionally to changes in the triangle.

### Adaptive Gaussian Splitting via EVSplitting

A further problem arises during deformation: large gaussians bound to small triangles scale disproportionately fast, producing visual artifacts in heavily deformed regions. To mitigate this, an adaptive splitting step was introduced at bind time.

The splitting condition is borrowed from prior work on mesh-aware gaussian splatting at train time: if a gaussian's largest dimension exceeds lambda multiplied by the circumcircle radius of its target triangle, it is split before binding. Since the original formulations of this condition are designed for use during training (where gradients can guide the split), a different mechanism is needed for post-hoc splitting in Blender. The addon uses the closed-form, visually consistent splitting method introduced in EVSplitting (SIGGRAPH Asia 2024), which provides an analytically exact decomposition of a gaussian into sub-gaussians. The alpha channel is maintained during splitting to correctly handle transparency.

[images of deformation artifacts from large gaussians before splitting]

### Multi-Scale Lambda Control (Global vs. Local)

An overly strict splitting threshold (lambda) can cause the total gaussian count to explode, particularly when the proxy mesh is dense. In practice, most regions of a splat do not require fine-grained deformability — only areas near joints or regions of high curvature benefit from smaller, more numerous gaussians. This mirrors conventional mesh-based workflows where artists add additional edge loops near elbows and joints for smoother deformation.

To address this, the addon supports two levels of lambda control. A lenient global lambda applies a conservative splitting threshold across the entire splat, while a strict local lambda can be applied to user-marked regions where high deformability is needed. This allows artists to concentrate computational cost where it matters, following the same intuition they already use when preparing meshes for skinning.

## Evidence

### Qualitative Comparison: Naive Binding vs. Priority Queue Assignment

The topology-aware priority queue assignment was developed specifically to handle cases where naive closest-triangle binding produces incorrect assignments. Regions with interleaved geometry such as fingers are a representative failure case for the naive approach. Evidence for the improvement is currently qualitative, based on visual inspection of deformed splats.

[images comparing deformation results: naive assignment vs. priority-queue assignment]

### Deformation Artifacts Before and After Gaussian Splitting


The effect of adaptive gaussian splitting is demonstrated on a splat of a toy arm, captured on a Samsung Galaxy S23 with fewer than 150 images. Without splitting, deformation of the bound splat produces visible artifacts — particularly stretching and bloating — in regions where large gaussians are bound to small triangles. After applying EVSplitting with a moderately strict local lambda of 1.5 and no global lambda, these artifacts are visibly reduced or eliminated.

[identical views of 3 different splats: original, non-deformed, deformed splat without splitting, deformed splat with splitting]

### Lambda Sensitivity

Preliminary observations indicate that the lambda threshold has a meaningful impact on both visual quality and gaussian count. A strict lambda produces finer gaussians and smoother deformation at the cost of increased splat density. A lenient lambda preserves the original gaussian count but may leave artifacts in high-deformation regions. The dual global/local lambda scheme is intended to offer a practical middle ground, though systematic evaluation of this trade-off remains future work.

### Demonstration on Casually Captured Data

All examples shown in this report were produced from a splat captured on a Samsung Galaxy S23 using fewer than 150 images. This is deliberately below the quality threshold of professional multi-camera capture rigs, in line with the design goal that the addon should be usable with amateurishly captured splats, not just studio-quality data.

[images of the casually captured splat alongside its proxy mesh in Blender]

## Minutes of Meetings

[to be appended in final report]
