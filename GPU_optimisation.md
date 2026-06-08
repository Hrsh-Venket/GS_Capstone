# GPU Optimisation

## Binding operation

1. Compute APSP for proxy mesh (Maybe Floyd-Warshall) to precompute for proxy-cost
2. Use label propogation instead of a priority queue as operations can be paralellised
3. At bind time and during each split, delete gaussians below an opacity threshold
4. Adjust values of existing gaussians and create 1 new gaussian instead of creating 2 new gaussians

## TODO

1. Bind_Gaussian_Splat_To_Proxy_Mesh
2. Compute_New_World_Positions
3. sna_viewport_render
4. sna_render_comp
5. extract_gaussian_data_from_evaluated_mesh
6. sna_c2_refresh_all
7. sna_texture_creation
8. apply_3dgs_transforms
9. export_mesh_object_as_3dgs_ply
10. mesh23dgs_3dgsfed
11. auto_generate_crop_object
12. render_remove_higher_sh_attributes
13. shader_system.sna_shader_system

