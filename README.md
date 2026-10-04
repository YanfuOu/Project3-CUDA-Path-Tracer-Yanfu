CUDA Path Tracer
================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 3**

* Yanfu Ou
* Tested on: Ubuntu 24.04LTS, AMD Ryzen 7 7840HS, Nvidia RTX 5070 Laptop GPU(8GB VRAM), 64GB DDR4 RAM


### Features Implemented
1. Contiguous memory by material type 
2. Stochastic sampled antialiasing

| No antialiasing | Antialiasing |
| --- | --- |
| ![No antialiasing](images/anti-alising/cornell-no-anti-alising.png) | ![Antialiasing](images/anti-alising/cornell-anti-alised.png) |
| ![No antialiasing](images/anti-alising/cornell-no-anti-alising-zoomed.png) | ![Antialiasing](images/anti-alising/cornell-anti-alised-zoomed.png) |
In the image with no anti-alising, the sphere's edge have very obvious stair casing effect. 

3. Depth of Field (2pt)

| Focused on Cube | Focused on Sphere |
| --- | --- |
| ![No antialiasing](images/depth-of-field/cornell-DOF-cube.png) | ![Antialiasing](images/depth-of-field/cornell-DOF-sphere.png) |

4. Refraction (2pt)
Blue Sphere in the back

| Front Left view | Inside Box Top View |
| --- | --- |
| ![No antialiasing](images/refraction-reflection/cornell-refraction-specular-blue-sphere-front.png) | ![Antialiasing](images/refraction-reflection/cornell-refraction-specular-blue-sphere-side.png) |

White Sphere in the Back
![Antialiasing](images/refraction-reflection/cornell-refraction-specular-og-sphere-front.png)

5. Direct Lighting (2pt)
Path Trace Depth of 1 

| No Direct Light | With Direct Lighting |
| --- | --- |
| ![No antialiasing](images/direct-lighting/cornell-direct-ray-no-depth1.png) | ![Antialiasing](images/direct-lighting/cornell-direct-ray-depth1.png) |

Path Trace Depth of 2 

| No Direct Light | With Direct Lighting |
| --- | --- |
| ![No antialiasing](images/direct-lighting/cornell-direct-ray-no-depth2.png) | ![Antialiasing](images/direct-lighting/cornell-direct-ray-depth2.png) |


6. Better Random Sequence (3pt)

| Iterations | With Random Sequence | Without Random Sequence |
| --- | --- | --- | 
| 8 iter | ![No antialiasing](images/halton-sampling/cornell-halton-8iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-8iter-3depth.png) |
| 16 iter | ![No antialiasing](images/halton-sampling/cornell-halton-16iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-16iter-3depth.png) |
| 64 iter | ![No antialiasing](images/halton-sampling/cornell-halton-64iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-64iter-3depth.png) |
| 128 iter | ![No antialiasing](images/halton-sampling/cornell-halton-128iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-128iter-3depth.png) |
| 256 iter | ![No antialiasing](images/halton-sampling/cornell-halton-256iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-256iter-3depth.png) |
| 512 iter | ![No antialiasing](images/halton-sampling/cornell-halton-512iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-512iter-3depth.png) |
| 1024 iter | ![No antialiasing](images/halton-sampling/cornell-halton-1024iter-3depth.png) | ![Antialiasing](images/halton-sampling/cornell-halton-no-1024iter-3depth.png) |



7. Russian Roulette (1pt) 
