**University of Pennsylvania, CIS 565: GPU Programming and Architecture,
Project 1 - Flocking**

* Bora Ersoy 
  *  [https://www.linkedin.com/in/bora-ersoy-0b7950212/]()
* Tested on: 13th Gen Intel(R) Core(TM) i7-13700HX, 2100 Mhz, 16 Core(s), 24 Logical Processor(s), RTX 4060 8GB AD107

### Naive Boids Simulation

![nicegif](https://github.com/user-attachments/assets/fac0965e-098c-4a6c-9a9b-3d3b76fdb917)



**Comparing Naïve, Scattered-Grid, and Coherent-Grid Implementations (CUDA)**

This section summarizes how performance changes with the number of boids, block configurations, grid coherence, and cell width. Results are based on measurements taken for N = 10k–400k boids and various block sizes.

<img width="645" height="392" alt="Screenshot 2025-11-27 133545" src="https://github.com/user-attachments/assets/6005fd08-70ac-4f95-b93a-9687f67fcc5a" />
<img width="635" height="414" alt="Screenshot 2025-11-27 133555" src="https://github.com/user-attachments/assets/1f4571b4-a649-4fc5-8ca1-8cc705bc0e4a" />
<img width="633" height="409" alt="Screenshot 2025-11-27 133601" src="https://github.com/user-attachments/assets/c8c32df6-baf0-4e7f-8859-a13cce839850" />

**Effect of increasing the number of boids**

As the number of boids increases, each implementation behaves differently.
In the naïve version, performance drops extremely fast. At 10k boids it runs at 400 FPS, but by 50k it falls to around 32 FPS. This happens because every boid checks every other boid, so the work grows with 
𝑁 squared
. Even small increases in N cause huge slowdowns.

The scattered uniform grid behaves almost linearly with N. It starts around 1700 FPS at 10k boids and gradually falls to 100 FPS at 200k. The spatial grid limits neighbor checks to nearby cells, so the cost grows roughly with density instead of the entire population.

The coherent grid shows the same near-linear trend but runs much faster overall. It gives around 700 FPS at 100k, 460 FPS at 200k, and 200 FPS at 400k. The memory layout is tightly packed, so cache and memory access patterns are far better than in the scattered grid.

**Effect of block size and block count**

Changing block size (64, 128, 160, 256) barely changed performance. For example, with 100k boids on the coherent grid, all tested block sizes produced about 700 FPS.

The kernels are memory-bound, so once occupancy is high enough, changing the block size does not significantly affect runtime. The GPU already has enough warps to hide memory latency, so larger or smaller blocks do not meaningfully improve speed. Only extreme or invalid block sizes would cause noticeable changes.

**Coherent grid performance improvement**

The coherent grid gave a very clear speedup compared to the scattered grid. Sorting the boids by cell and storing them contiguously greatly improves memory locality. Neighbor data ends up close together in memory, so global memory reads become coalesced and cache hits increase. Because of that, the coherent grid behaves the same algorithmically as the scattered grid but with a much higher FPS. This is exactly what would be expected from improved memory coherence.

**Effect of cell width and checking 27 vs 8 neighboring cells**

Decreasing cell width improved performance, giving about a 200-FPS boost in the coherent grid tests. Although 27 cells sounds like more work than 8, the number of actual boids per cell matters more. Smaller cells reduce the number of boids inside each one. Even though more cells are checked overall, each cell contains fewer boids, so the total number of neighbor comparisons can drop.

Because of the coherent layout, checking multiple small, contiguous memory ranges can be faster than checking a few large, sparse ranges. The cost depends on density and memory locality, not only on cell count. This is why the 27-cell version is not automatically slower and in these tests actually ran faster with the right cell width.

