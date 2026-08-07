# Lid-Driven Cavity Flow — Vortex Dynamics Across Re 100–3200

A CFD study built in ANSYS Fluent, examining how recirculation structure in a lid-driven cavity evolves across Reynolds number and how cavity aspect ratio changes the flow, validated against the Ghia, Ghia & Shin (1982) benchmark.

I built this as a self-directed portfolio project to get hands-on with a classic CFD benchmark problem end to end — geometry, meshing, solving, and validating against published data, rather than just following a tutorial.

 Table of Contents
- [What Was Done](#what-was-actually-done)
- [Setup / Software Used](#setup--software-used)
- [Reproducing the Results](#reproducing-the-results)
- [Results](#results)
- [Method, Briefly](#method-briefly)
- [Repository Structure](#repository-structure)
- [Author](#author)

## What was actually done

- Simulated 2D, steady, laminar lid-driven cavity flow in ANSYS Fluent across **Re = 100, 400, 1000, and 3200**
- Ran the full Re sweep on two geometries: a **square cavity** (0.1 m × 0.1 m) and a **rectangular cavity** (0.1 m × 0.2 m, aspect ratio 1:2)
- Validated centerline velocity profiles against the Ghia, Ghia & Shin (1982) benchmark data
- Controlled Re by fixing cavity width and lid speed and varying fluid viscosity, rather than changing geometry or velocity between runs
- Diagnosed and fixed a convergence stall at Re = 3200 by lowering under-relaxation factors instead of just extending iteration count
- Extracted and exported centerline velocity data (u vs. y, v vs. x) for every case, using ordered line-rake surfaces to avoid the jagged/unsorted plotting issue that shows up if node order isn't handled correctly
- Attempted to resolve secondary corner vortices at Re = 3200 via tightened contour ranges and point probing — got close, but couldn't get an unambiguous confirmation with the mesh resolution and time available (see Limitations)

## Setup / Software Used

* ANSYS DesignModeler (geometry)
* ANSYS Meshing — MultiZone method, wall-biased structured quad mesh
* ANSYS Fluent 2025/2026 R1, Student version (solver)
* Fluent's native XY-Plot and Contour tools for post-processing and benchmark comparison

## Reproducing the Results

1. Open ANSYS Workbench → Fluid Flow (Fluent)
2. Build the geometry as a flat 2D rectangle in DesignModeler — 0.1 m × 0.1 m for the square case, 0.1 m × 0.2 m for the aspect-ratio case — and convert to a surface body (Concept → Surfaces From Sketches)
3. Mesh using the MultiZone method with edge bias toward all four walls to resolve near-wall and corner gradients
4. In Fluent: set 2D Space to Planar, Viscous Model to Laminar, and assign the top edge as a moving wall (lid) at 1 m/s; the remaining three edges stay as stationary no-slip walls
5. Set fluid density to 1 kg/m³ and adjust viscosity to hit each target Re (see table below), keeping cavity width and lid speed fixed
6. Solve with SIMPLE, second-order upwind momentum, residual target 1×10⁻⁶ on continuity and both velocity components
7. For Re = 3200, if continuity stalls before reaching the target, lower under-relaxation (pressure ≈ 0.2, momentum ≈ 0.4) and continue iterating rather than restarting
8. Extract centerline velocity profiles using Line/Rake surfaces at the geometric center, with **Order Points** enabled in the XY-Plot dialog — without this the plotted curve comes out jagged since Fluent samples nodes in mesh order, not spatial order

## Results

**Reynolds number control (fixed ρ = 1 kg/m³, L = 0.1 m, U = 1 m/s)**

| Re | Viscosity (Pa·s) |
|---|---|
| 100 | 0.001 |
| 400 | 0.00025 |
| 1000 | 0.0001 |
| 3200 | 0.00003125 |

**Square cavity** — centerline u- and v-velocity profiles were extracted for all four Re and are included in `results/velocity_profiles/`, alongside a qualitative comparison against Ghia et al. (1982). As Re increases, the velocity excursions deepen and the vortex core migrates toward the cavity center, consistent with the published trend.

**Rectangular cavity (1:2)** — the primary vortex stays confined to roughly the upper half of the domain, with a large, weakly-moving region below it, consistent with published behavior for elongated cavities.

Full contour plots and residual convergence history for every case are in `results/contours/` and `results/residuals/` respectively.

## Method, briefly

- **Geometry:** Flat 2D surface bodies — no extrusion, no third dimension. Square (0.1 × 0.1 m) and rectangular (0.1 × 0.2 m) cavities, built and meshed as separate Workbench systems.
- **Mesh:** Structured quadrilateral mesh via MultiZone, with bias factors applied at all four edges to cluster cells near the walls and corners, where the interesting vortex structure lives.
- **Solver:** Pressure-based, steady, laminar, SIMPLE scheme, second-order upwind momentum, residuals converged to 10⁻⁶ on continuity and both velocity components.
- **Reynolds number control:** rather than changing geometry or lid speed, Re was set purely by adjusting fluid viscosity — this kept the mesh and boundary conditions identical across all four cases per geometry, isolating Re as the only variable.
- **Velocity profile extraction:** vertical and horizontal centerline Line/Rake surfaces, sampled with Order Points enabled and the correct Plot Direction vector for each line orientation — getting this wrong the first few times produced flat or jagged plots before I traced it back to unordered node sampling.
- **Convergence at Re = 3200:** default under-relaxation factors caused continuity to plateau around 10⁻³ regardless of iteration count. Reducing pressure and momentum under-relaxation and continuing the run (rather than restarting from scratch) resolved this.

## Limitations

At Re = 3200, secondary and (theoretically) tertiary corner vortices are expected based on prior literature, but I wasn't able to get a fully unambiguous visual confirmation of them with the mesh resolution and post-processing time available — tightening the contour range down to the order of 10⁻⁸–10⁻⁹ kg/s and probing individual points got close, but not a clean result I'd want to present as conclusive. I'm noting this directly rather than glossing over it. The primary vortex dynamics — the main physics this study set out to capture — are fully resolved and validated across all cases and both geometries.

A formal three-level Grid Convergence Index (GCI) study via Richardson extrapolation was planned but not completed in this version; the production mesh was checked for qualitative independence (skewness, orthogonal quality) rather than run through the full GCI procedure.

## Repository structure

```
├── README.md
├── geometry/
│   └── (DesignModeler files, square and rectangular cavities)
├── mesh/
│   └── (exported FLUENT .msh files, both geometries)
├── results/
│   ├── contours/            → stream function contours, all Re, both geometries
│   ├── velocity_profiles/   → exported centerline velocity data (.xy) and plots
│   └── residuals/           → convergence history for every run
└── case_files/
    └── lid driven cavity
    └── 2nd lid driven cavity
```

## Author

**Pratyush Dash**

B.Tech Chemical Engineering, KIIT University, Bhubaneswar
