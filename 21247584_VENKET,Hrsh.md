# Interim Report: Topology-Aware Deformation of Gaussian Splats

Author: VENKET, Hrsh

Student ID: 21247584 

The objective of this work is to create a useable system by which Gaussian splats taken from a non-professional setup can be deformed while minimising artificats. My solution to this is bind gaussian splats to a proxy mesh in a topologically aware manner so when an artist can simply deform the proxy mesh to deform a splat without creating artifacts

### Demonstration on Casually Captured Data

All examples shown in this report were produced from a splat captured on a Samsung Galaxy S23 using fewer than 150 images. This is deliberately below the quality threshold of professional multi-camera capture rigs, in line with the design goal that the addon should be usable with amateurishly captured splats, not just studio-quality data. Below are screenshots of this naively caputured splat, and one arm isolated as a demonstration of this approach. The arm is shown using a representation of the splat (rendered as vertices at gaussian centers) and a proxy mesh of the same. The splat and proxy mesh are generated using software by KIRI Engine, and the tooling I have built is onto a fork by the same company.

![](assets/full_gs_sample.png){height=4in}
![](assets/gs+proxy_mesh.png){height=4in}

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

The splitting condition is borrowed from prior work on mesh-aware gaussian splatting at train time: if a gaussian's largest dimension exceeds a lambda multiplied by the circumcircle radius of its target triangle, it is split before binding. Since the original formulations of this condition are designed for use during training (where gradients can guide the split), a different mechanism is needed for post-hoc splitting in Blender. The addon uses the closed-form, visually consistent splitting method introduced in EVSplitting (SIGGRAPH Asia 2024), which provides an analytically exact decomposition of a gaussian into sub-gaussians. The alpha channel is maintained during splitting to correctly handle transparency.

Below are the closed-form solutions for splitting these gaussians. For convenience of notation, we can think of them as left and right gaussians.

The positions $\mu_{l}$ and $\mu_{r}$ of the new gaussians defined as follows:
$$\mu_l = \mu_0 - \frac{\mathbf{L}_0 D}{\tau C_l}$$
$$\mu_r = \mu_0 + \frac{\mathbf{L}_0 D}{\tau C_r}$$

Their alpha channel or opacity are defined: 
$$\alpha_k = \alpha_0 C_k, \quad k \in \{l, r\}$$

Similarly, their covariance matrices $\Sigma_{l}$ and $\Sigma_{r}$ are defined:
$$\Sigma_l = \Sigma_0 + \frac{\mathbf{L}_0 \mathbf{L}_0^T}{\tau^2} \left( \frac{d_0 D}{\tau C_l} - \frac{D^2}{C_l^2} \right)$$
$$\Sigma_r = \Sigma_0 - \frac{\mathbf{L}_0 \mathbf{L}_0^T}{\tau^2} \left( \frac{d_0 D}{\tau C_r} + \frac{D^2}{C_r^2} \right)$$

Where the auxiliary quantities are defined as follows. $C_l$ and $C_r$ are the spatial weights representing the fraction of the original gaussian's probability mass on each side of the splitting plane:

$$C_l = C_r =  \frac{1}{2} \left(1 - \operatorname{erf}\left(\frac{d_0}{\sqrt{2}\tau}\right)\right)$$

where $\operatorname{erf}$ is the Gauss error function. When the splitting plane passes through the gaussian centre, $C_l = C_r = 0.5$ and the split is symmetric. As the plane moves to one side, the weight on that side approaches zero and the other approaches one.

$D$ is a density factor equal to the value of the one-dimensional unit gaussian pdf evaluated at the normalised distance of the splitting plane from the gaussian centre:

$$D = \frac{1}{\sqrt{2\pi}} \exp\left(-\frac{d_0^2}{2\tau^2}\right)$$

This term governs the magnitude of the position and covariance corrections. When the plane is far from the gaussian centre, $D \to 0$ and the child gaussians converge to the original, reflecting the fact that no meaningful split occurs.

$\mathbf{L}_0$ is the projection of the original covariance matrix onto the splitting plane normal $\mathbf{n}$:

$$\mathbf{L}_0 = \Sigma_0 \mathbf{n}$$

This vector captures both the spread of the gaussian along $\mathbf{n}$ and the correlations between the split direction and the remaining axes. It ensures the position and covariance updates are correct for anisotropic gaussians whose principal axes are not aligned with the splitting plane.

$\tau$ is the standard deviation of the original gaussian along the splitting plane normal:

$$\tau = \sqrt{\mathbf{n}^T \Sigma_0 \mathbf{n}}$$

Finally, $d_0$ is the signed distance from the original gaussian centre $\mu_0$ to the splitting plane $P(\mathbf{n}, d)$:

$$d_0 = \mathbf{n} \cdot \mu_0 + d$$

The solution is derived by requiring each child gaussian to independently conserve the zeroth, first, and second-order moments of the original gaussian's opacity distribution over its own half-space. This yields a system of integral tensor equations with a unique closed-form solution. The SH coefficients for view-dependent colour are copied directly to both children, as the positional offset between child and parent is negligible relative to typical camera distances.

| Highlighted artifacts | Deformed splat (no splitting) |
|:---------------------:|:-----------------------------:|
| ![](assets/Views/cropped/artifacts/no_duplicaates_toy_arm_v1_0076_color.png){height=1.9in} | ![](assets/Views/cropped/no_duplicaates_toy_arm_v1_0076_color.png){height=1.9in} |
| ![](assets/Views/cropped/artifacts/no_duplicaates_toy_arm_v2_0076_color.png){height=1.9in} | ![](assets/Views/cropped/no_duplicaates_toy_arm_v2_0076_color.png){height=1.9in} |
| ![](assets/Views/cropped/artifacts/no_duplicaates_toy_arm_v3_0076_color.png){height=1.9in} | ![](assets/Views/cropped/no_duplicaates_toy_arm_v3_0076_color.png){height=1.9in} |

: Deformation artifacts produced when large gaussians are bound to small triangles, shown on the deformed splat before splitting. The left column marks regions of stretching and bloating with magnified call-outs; the right column shows the corresponding unannotated view. Rows are three camera views of the same splat.

### Multi-Scale Lambda Control (Global vs. Local)

An overly strict splitting threshold (lambda) can cause the total gaussian count to explode, particularly when the proxy mesh is dense. In practice, most regions of a splat do not require fine-grained deformability -- only areas near joints or regions of high curvature benefit from smaller, more numerous gaussians. This mirrors conventional mesh-based workflows where artists add additional edge loops near elbows and joints for smoother deformation.

To address this, the addon supports two levels of lambda control. A lenient global lambda applies a conservative splitting threshold across the entire splat, while a strict local lambda can be applied to user-marked regions where high deformability is needed. This allows artists to concentrate computational cost where it matters, following the same intuition they already use when preparing meshes for skinning. As pictured below, the UI uses existing blender selection tools to mark regions with a strict lambda. For future iterations of this, as needed, it may be useful to allow users to 'paint' a lambda onto the splat or otherwise mark different local lambda values.

![](assets/strict_lambda_ui.png)

### Design Goals

A large aspect of designing this plugin has been integrating it with existing blender tools. Although there are separate buttons to mark lambda and do certain gassian-splat related operations, all selection is done using regular blender UI in edit mode. Further, the artist can use all existing methods for mesh deformation to apply similar operations to gaussians. Control remains squarely with them for how they want to use different operations and adjust the Splat by adding or removing gaussians.


## Evidence

### Qualitative Comparison: Naive Binding vs. Priority Queue Assignment

The topology-aware priority queue assignment was developed specifically to handle cases where naive closest-triangle binding produces incorrect assignments. Regions with interleaved geometry such as fingers are a representative failure case for the naive approach. Evidence for the improvement is currently qualitative, based on visual inspection of deformed splats.

### Deformation Artifacts Before and After Gaussian Splitting

The effect of adaptive gaussian splitting is demonstrated on a splat of a toy arm, captured on a Samsung Galaxy S23 with fewer than 150 images. Without splitting, deformation of the bound splat produces visible artifacts — particularly stretching and bloating — in regions where large gaussians are bound to small triangles. After applying EVSplitting with a moderately strict local lambda of 1.5 and no global lambda, these artifacts are visibly reduced or eliminated.

| Original (non-deformed) | Deformed, without splitting | Deformed, with splitting |
|:-----------------------:|:---------------------------:|:------------------------:|
| ![](assets/Views/cropped/base_toy_arm_v1_0076_color.png){height=1.9in} | ![](assets/Views/cropped/no_duplicaates_toy_arm_v1_0076_color.png){height=1.9in} | ![](assets/Views/cropped/full_toy_arm_v1_0076_color.png){height=1.9in} |
| ![](assets/Views/cropped/base_toy_arm_v2_0076_color.png){height=1.9in} | ![](assets/Views/cropped/no_duplicaates_toy_arm_v2_0076_color.png){height=1.9in} | ![](assets/Views/cropped/full_toy_arm_v2_0076_color.png){height=1.9in} |
| ![](assets/Views/cropped/base_toy_arm_v3_0076_color.png){height=1.9in} | ![](assets/Views/cropped/no_duplicaates_toy_arm_v3_0076_color.png){height=1.9in} | ![](assets/Views/cropped/full_toy_arm_v3_0076_color.png){height=1.9in} |

: Three identical camera views (rows) of the toy-arm splat under each condition. The deformed splat without gaussian splitting shows stretching and bloating near the elbow joint; with EVSplitting applied (local lambda 1.5, no global lambda) these artifacts are visibly reduced.

### Lambda Sensitivity

Preliminary observations indicate that the lambda threshold has a meaningful impact on both visual quality and gaussian count. A strict lambda produces finer gaussians and smoother deformation at the cost of increased splat density. A lenient lambda preserves the original gaussian count but may leave artifacts in high-deformation regions. The dual global/local lambda scheme is intended to offer a practical middle ground, though systematic evaluation of this trade-off remains future work.

## Future Work

Below is a list of work I intend to work on to see this work to completion. 

- [ ] Optimisation:
    - [ ] Improve performance of the binding and rendering processes to run on GPU. Ideas include:
        - [ ] Remove transparent gaussians (/highly translucent)
        - [ ] Pre-compute splits at different lambdas before the user has decided where to do binding
        - [ ] Detection of problematic gaussians given a particular armature
    - [ ] Improve UI elements to integrate more cleanly into blender
    - [ ] Fix small bugs in viewport rendering 
- [ ] Evaluation: 
    - [ ] A way to systematically evaluate artifacts in:
        - [ ] Skeleton based methods (LBS, DQS)
        - [ ] Non-rigid deformation
    - [ ] Metrics
        - [ ] Against sythetic examples
        - [ ] Cage based methods
- [ ] Relighting of Gaussians (this can be an orthogonal problem, although spherical harmonics are being updated using the existing code so this might be fairly straightforward)


## Minutes of Meetings

[to be appended in final report]
