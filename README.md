CUDA Path Tracer - Joyful Snowman
================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 3**

* Yanfu Ou
* Tested on: Ubuntu 24.04 LTS, AMD Ryzen 7 7840HS, Nvidia RTX 5070 Laptop GPU (8 GB VRAM), 64 GB DDR4 RAM

![Joyful snowman](images/snowman-in-brick-FINAL.png)

A CUDA path tracer built around an artistic reimagination of a snowman in a colorful house. Fresnel reflection and refraction shows the snowman's head. The mirror like specular sphere demonstrates the snowman's upper body. Last, but not the least, the crimson red illistrates the snowman's lower body! The snowman sits in a joyful colored room defined by gold, blue, and pink walls!   

The renderer is a combination of diffuse and specular shading, refraction with Fresnel, depth of field, direct lighting, Halton sampling, Russian roulette, stochastic antialiasing, and shading paths sorted into contiguous memory by material type. Every feature is a `0`/`1` switch in `src/featureToggles.h`. Flip a switch and rebuild to see the difference!




### 1. Depth of field

The camera is modeled as a thin len. The ray origin is shifted to a random point on a disk of radius `0.15`, and the ray is aimed at the focal plane so that plane stays sharp. In the snowman scene, the focal distance is `10.5`, which is the distance from the camera at `z = 10.5` to the specular sphere at `z = 0`. As you can see, the specular upperbody of the snowman is in focused, while the brick wall in the back is out of focus and appear to be blurry. However, when we set the focal distance to `z = 15.27`, we can see the vice versa effect. Toggling `DEPTH_OF_FIELD` switches back to a pinhole.

| Focused on the snowman | Focused on the blue backwall |
| --- | --- |
| ![Focused on snowman](images/depth-of-field/snowman-DOF-specular-sphere.png) | ![Focused on snowman](images/depth-of-field/snowman-DOF-blue-wall.png)  |
| ![Focused on snowman](images/depth-of-field/snowman-DOF-specular-sphere-zoomed.png) | ![Focused on snowman](images/depth-of-field/snowman-DOF-blue-wall-zoomed.png)  |

_above images are generated at depth = 8 and 1k iters_

For a more clear demo, you can see the same effect running in the cornell box below between a sphere and a cube. 

| Focused on the cube | Focused on the sphere |
| --- | --- |
| ![Depth of field, cube](images/depth-of-field/cornell-DOF-cube.png) | ![Depth of field, sphere](images/depth-of-field/cornell-DOF-sphere.png) |

_above images are generated at depth = 8 and 5k iters_


#### Hypothethical CPU Comparison
The lens adds two random numbers and makes primary rays less coherent, because neighbors no longer share an origin. That hurts ray-packet style traversal, which this tracer does not use. The intersection loop is still one ray per thread, so the GPU handles the incoherent origins well. A CPU implementation would trace the same rays more slowly. A smaller aperture, or a lookup of focal distance from the scene file, would make the effect easier to tune without a rebuild.

### 2. Refraction

Glass uses Snell's law through `glm::refract` and Schlick's approximation to choose reflection or transmission. The index of refraction is the material `IOR` (1.5 in these scenes). Entering glass uses `1 / IOR`. Leaving glass uses `IOR`. Total internal reflection, and a Fresnel sample above the reflection probability, both reflect. `REFRACTION` turns glass into a perfect mirror.

The glass head of the snowman shows this effect: the room bends through the sphere, and the rim reflects.

| Left view | Right view |
| --- | --- |
| ![Refraction, left](images/refraction-reflection/snowman-refraction-left-view.png) | ![Refraction, right](images/refraction-reflection/snowman-refraction-right-view.png) |

_above images are generated at depth = 8 and 1k iters_

For a more clear demo, you can see the same effect running in the cornell box below with a glass sphere, diffuse sphere, and a reflective cube. 

| Front view | Side view |
| --- | --- |
| ![Refraction, front](images/refraction-reflection/cornell-refraction-specular-blue-sphere-front.png) | ![Refraction, side](images/refraction-reflection/cornell-refraction-specular-blue-sphere-side.png) |

_above images are generated at depth = 8 and 5k iters_


#### Hypothethical CPU Comparison
Refraction does not add primitives. It adds a branch in the shade kernel and paths that continue through the sphere instead of stopping at the surface, so those pixels need the full trace depth. On the GPU that branch diverges only for threads that hit glass. A warp that is mostly diffuse stays on the diffuse path. A CPU version has no warp divergence, but it also has no thousands of pixels in flight. The next step would be a rough dielectric, so transmission is not a single perfect ray.

### 3. Direct lighting (2 pt)

On the last bounce of a diffuse path, the tracer samples a random point on a random emissive quad and sends a shadow ray there. The contribution is the Lambertian term times the light's cosine, divided by distance squared and by the area pdf. If that ray hits anything else, the sample is black. Specular and glass paths do not take this branch, because a next-event ray is not the reflected or refracted direction. `DIRECT_LIGHTING` disables the shadow ray.

#### Depth 1:

| No direct light | Direct light |
| --- | --- |
| ![Depth 1, no direct light](images/direct-lighting/snowman-direct-ray-no-depth1.png) | ![Depth 1, direct light](images/direct-lighting/snowman-direct-ray-depth1.png)| 
| ![Depth 1, no direct light](images/direct-lighting/cornell-direct-ray-no-depth1.png) | ![Depth 1, direct light](images/direct-lighting/cornell-direct-ray-depth1.png) |


At depth 1 the snowman scene without direct light is black except for the ceiling panel. This is because the camera ray ends on the first hit, so the bricks, the floor, and the crimson body never get a second bounce that could find the light.

With direct light, that one shadow ray is enough to color the pink, gold, and blue walls and the floor. The snowman stays a dark silhouette. The glass head and the mirror body are not diffuse, so they do not take a next-event sample. The crimson sphere only picks up a little red on the top that faces the lamp. The dark patch on the floor under the snowman is the shadow ray being blocked.

The cornell box demo show the same principals. However, the snowman demo appear to show this principal more clearly. 

#### Depth 2:

| No direct light | Direct light |
| --- | --- |
| ![Depth 2, no direct light](images/direct-lighting/snowman-direct-ray-no-depth2.png) | ![Depth 2, direct light](images/direct-lighting/snowman-direct-ray-depth2.png) |
| ![Depth 2, no direct light](images/direct-lighting/cornell-direct-ray-no-depth2.png) | ![Depth 2, direct light](images/direct-lighting/cornell-direct-ray-depth2.png) |


At depth 2 the snowman scene without direct light is no longer black. The second bounce can reach the ceiling light, so the brick walls, the floor, and a hint of red on the crimson body show up. Most of those bounces still miss the small lamp, so the walls stay dim and speckled. The glass head and the mirror body only flash white where the reflection happens to point at the light.

With direct light, that second bounce is aimed at the lamp. The pink, gold, and blue bricks, the floor, and the crimson body fill in, and the ceiling picks up light too. The mirror body reflects the colored walls beside it instead of staying a black sphere with one white glint. The floor still has a shadow under the snowman, but the rest of the ground is evenly lit.

The cornell box demo show the same principals. However, the snowman demo appear to show this principal more clearly. 

#### Hypothethical CPU Comparison
Each diffuse path that reaches its last bounce traces one extra ray. That is one more full pass over the geometry. The GPU still does this as one thread per path, so the extra pass scales with the number of live paths. A CPU version would walk the same shadow rays serially. Multiple importance sampling, or sampling the light on every diffuse bounce instead of only the last one, would cut noise further.

### 4. Halton sampling (3 pt)

Random numbers come from a Halton sequence, one prime per dimension, with a per-pixel Cranley-Patterson shift so neighboring pixels do not share the same sequence. Past 32 dimensions the sample falls back to a hash. Camera jitter, the lens, diffuse bounces, Fresnel, direct-light selection, and Russian roulette all consume this stream in order. Turning a feature off still advances the dimensions it would have used, so the remaining features stay on the same coordinates. `HALTON_SAMPLING` falls back to `thrust`'s uniform generator.

#### Snowman, depth 3:

| Iterations | Halton | Uniform random |
| --- | --- | --- |
| 8 | ![Snowman Halton, 8](images/halton-sampling/snowman-halton-8iter-3depth.png) | ![Snowman random, 8](images/halton-sampling/snowman-halton-no-8iter-3depth.png) |
| 16 | ![Snowman Halton, 16](images/halton-sampling/snowman-halton-16iter-3depth.png) | ![Snowman random, 16](images/halton-sampling/snowman-halton-no-16iter-3depth.png) |
| 64 | ![Snowman Halton, 64](images/halton-sampling/snowman-halton-64iter-3depth.png) | ![Snowman random, 64](images/halton-sampling/snowman-halton-no-64iter-3depth.png) |
| 128 | ![Snowman Halton, 128](images/halton-sampling/snowman-halton-128iter-3depth.png) | ![Snowman random, 128](images/halton-sampling/snowman-halton-no-128iter-3depth.png) |
| 256 | ![Snowman Halton, 256](images/halton-sampling/snowman-halton-256iter-3depth.png) | ![Snowman random, 256](images/halton-sampling/snowman-halton-no-256iter-3depth.png) |

_above images are generated at depth 3_

On the snowman, Halton fills the pixel more evenly, so the same iteration count is less blotchy. The gap is largest at 8 to 64 samples and shrinks once both sequences have enough samples.

At 8 iterations both images are mostly noise, but they are different kind of noise. Uniform random looks like static: pink, gold, and blue specks are scattered with no structure. Additionally, the brick courses are hard to find. Halton noise is more ordered. The floor, the mortar lines, and the crimson body already read as surfaces instead of a cloud of points. 

By 64 iterations the Halton frame has a fine grain and the mirror is picking up pink and gold. The uniform frame is still blotchy, especially on the gold wall and the floor. 

At 256 iterations Halton has cleaned up the brick joints, the floor, and the glass head, which shows the room through the sphere. Uniform random at the same count still has a coarse grain across the walls and floor, and the glass head stays a dark sphere with a lamp highlight. The mirror and the crimson body settle in both, because those pixels are less dependent on a random diffuse bounce. The bricks and the floor are where the sequence shows up.

#### Cornell Box Depth 3
| Iterations | Halton | Uniform random |
| --- | --- | --- |
| 8 | ![Halton, 8](images/halton-sampling/cornell-halton-8iter-3depth.png) | ![Random, 8](images/halton-sampling/cornell-halton-no-8iter-3depth.png) |
| 16 | ![Halton, 16](images/halton-sampling/cornell-halton-16iter-3depth.png) | ![Random, 16](images/halton-sampling/cornell-halton-no-16iter-3depth.png) |
| 64 | ![Halton, 64](images/halton-sampling/cornell-halton-64iter-3depth.png) | ![Random, 64](images/halton-sampling/cornell-halton-no-64iter-3depth.png) |
| 128 | ![Halton, 128](images/halton-sampling/cornell-halton-128iter-3depth.png) | ![Random, 128](images/halton-sampling/cornell-halton-no-128iter-3depth.png) |
| 256 | ![Halton, 256](images/halton-sampling/cornell-halton-256iter-3depth.png) | ![Random, 256](images/halton-sampling/cornell-halton-no-256iter-3depth.png) |

The effects on the glass sphere, diffused sphere, and reflective box is similar in the cornell box. 

#### Hypothethical CPU Comparison
The radical inverse is a short integer loop per sample. It is more arithmetic than a single uniform draw, and it is a poor fit for a warp if each thread stops the loop at a different sample index. The images still get cleaner faster, so the extra math pays for itself in fewer iterations. A CPU version has the same arithmetic and none of the warp issue, but it cannot shade the whole image at once. A Sobol sequence, or a precomputed sample table in constant memory, would remove the per-sample loop.

### 5. Russian roulette (1 pt)

After the third bounce, a path survives with probability `q`, where `q` is its brightest throughput channel clamped to `[0.05, 1]`. Survivors are scaled by `1 / q`, so the estimate stays unbiased. Paths that fail the test are terminated. Direct-light rays are not eligible. `RUSSIAN_ROULETTE` disables the test and lets paths run to `DEPTH`.

#### Hypothethical CPU Comparison
This does not change the final image's expected brightness. It shortens paths whose remaining color is already small, which matters in a closed box where rays would otherwise bounce until the depth limit. The GPU benefits directly: `thrust::remove_if` drops the dead paths, so later intersection launches have fewer threads. A CPU version would skip those rays too, but the savings show up as less serial work rather than a smaller kernel launch. Starting roulette one bounce earlier, or using the path's luminance instead of the max channel, would terminate more aggressively.

## Core features

### Cosine-weighted diffuse BSDF

A diffuse hit multiplies the path throughput by the material color and scatters a new ray from the intersection. The direction is cosine-weighted over the hemisphere around the normal: `u1` sets the polar angle with `sqrt(u1)`, and `u2` sets the azimuth. Brighter samples land closer to the normal, which is the Lambertian distribution, so the bounce does not need an extra cosine weight in the estimator. The brick walls, the floor, the ceiling, and the crimson snowman body all use this.

#### Hypothethical CPU Comparison
Diffuse paths are the noisy ones. Each bounce can go anywhere in the hemisphere, so most rays miss the small ceiling light unless direct lighting picks it for them. The hemisphere sample is a few square roots and a cross product, which is nothing next to intersecting every brick. On the GPU, a warp of diffuse hits stays together and runs the same math. A CPU version does the identical sample per ray and just gets through fewer rays. An importance sample that also knows about the light, which is what the direct-lighting feature adds on the last bounce, is the useful next step. Sampling the light on every diffuse bounce would converge faster than waiting for depth 1.

### Perfect specular reflection

A mirror hit reflects the incoming ray with `glm::reflect` and multiplies throughput by the material color. The new origin is nudged `0.001` along the reflected direction so the next trace does not hit the same surface. A perfect mirror consumes no random direction. Under Halton sampling it still advances two dimensions, so a later diffuse bounce stays on the coordinates it would have used if the mirror had been diffuse. The snowman's upper body is this material. With `REFRACTION` off, the glass head takes this path too and becomes a mirror.

#### Hypothethical CPU Comparison
The reflection is a fixed direction, so a specular pixel is sharp after one sample. The cost shows up in path length: the mirror keeps the path alive and sends it somewhere else in the room, often into the bricks or the glass. On the GPU, mirror hits diverge from diffuse hits in the same warp, which is why sorting by material helps. A CPU shader pays the same reflection math without that divergence, and it traces the continued rays one at a time. A roughness parameter, so the reflection is a lobe instead of a single ray, would be the next version of this BSDF.

### Stochastic sampled antialiasing

Each camera ray is jittered to a random point inside its pixel, so edges are averaged across samples instead of snapped to the pixel grid. Without jitter, the sphere silhouette looks like a staircase. With jitter, the same edge softens as iterations accumulate.`STOCHASTIC_ANTIALIASING` turns the jitter off and sends the ray through the pixel center.

| No antialiasing | Antialiasing |
| --- | --- |
| ![No antialiasing](images/anti-alising/snowman-anti-alised-no.png) | ![Antialiasing](images/anti-alising/snowman-anti-alised.png) |
| ![No antialiasing, zoomed](images/anti-alising/snowman-anti-alising-no-zoomed.png) | ![Antialiasing, zoomed](images/anti-alising/snowman-anti-alising-zoomed.png) |

For a more clear demo, here's a sphere running in cornell box. 

| No antialiasing | Antialiasing |
| --- | --- |
| ![No antialiasing](images/anti-alising/cornell-no-anti-alising.png) | ![Antialiasing](images/anti-alising/cornell-anti-alised.png) |
| ![No antialiasing, zoomed](images/anti-alising/cornell-no-anti-alising-zoomed.png) | ![Antialiasing, zoomed](images/anti-alising/cornell-anti-alised-zoomed.png) |



#### Hypothethical CPU Comparison
The extra cost is two random numbers per camera ray and a slightly less coherent set of primary rays. Intersection and shading dominate, so the jitter itself is cheap. On a GPU, neighboring threads already trace different pixels, and the jitter does not add much divergence. A CPU version would do the same math per ray and would not get the pixel-parallel throughput. A stratified or blue-noise pattern inside the pixel would converge with less noise than independent jitter.

### Sorting paths by material

Before shading, intersections and their paths are sorted by material id with `thrust::sort_by_key`. Threads in a warp are then more likely to take the same BSDF branch: diffuse, specular, or glass. `SORT_BY_MATERIAL` shades in pixel order instead.

#### Hypothethical CPU Comparison
The image does not change. The sort is an extra device-wide pass every bounce. It pays off when the shade kernel's branches are expensive and the scene mixes materials, which this scene does (brick, mirror, glass, light). On a GPU, divergent warps serialize those branches, so grouping equal materials keeps the warp together. A CPU shader does not have warps, so the same sort would mostly add overhead. Sorting once into separate material queues, instead of re-sorting the whole buffer every bounce, would drop the sort cost. A bounding volume hierarchy would matter more for the brick scene than another shading tweak, because intersection still tests every cube.

### Stream compaction

After shading, any path with `remainingBounces == 0` is removed with `thrust::remove_if`. A path reaches zero when it hits a light, misses the scene, fails Russian roulette, or runs out of depth. The next intersection launch only covers the paths still alive, and the bounce loop stops when none remain.

#### Hypothethical CPU Comparison
Without compaction, every pixel would keep a thread through the full trace depth, including pixels that already hit the ceiling light on bounce one. The brick scene makes that waste obvious: most primary rays hit a wall and continue, but rays that find the light, or that die in the glass, should not occupy a lane on the next bounce. `thrust::remove_if` is a device-wide stream compaction, so the GPU spends a prefix-sum-style pass to pack the survivors, then launches a smaller kernel. A CPU loop would just skip dead rays with a branch, which is cheaper per ray and slower overall because the live rays are still serial. A shared-memory compaction, or compacting before shading as well as after, would avoid shading lanes that already missed.
