# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A browser-based 3D fire fighting game built as a single HTML file (`forest_fire.html`). The player controls a fire engine to spray water and save a procedurally generated forest from spreading fire. Uses Three.js for 3D rendering and a voxel-based fluid dynamics simulation for fire behavior.

## Running

Open `forest_fire.html` directly in a modern browser. No build step, package manager, or local server required. Three.js is loaded from CDN (v0.160.0).

## Architecture

The entire application lives in `forest_fire.html`, organized into these sections:

**Simulation engine** (lines ~94-165): A 48x48x48 voxel grid (`N=48`) storing temperature (`temp`) and fuel (`fuel`) arrays. Fire spreads via heat diffusion using a simplified Laplacian, with buoyancy-driven advection. Key constants: `IGN_TEMP=0.22` (ignition threshold), `DIFFUSION=0.0003`, `SUBSTEPS=3` per frame. `simStep()` runs the physics; `applyWater()` handles fire suppression.

**Three.js scene** (lines ~167-274): Custom GLSL shaders for fire particles (additive blending, up to 40k particles). Tree canopy rendered as green point clouds (up to 50k particles). Water spray is a separate particle system (300 particles). OrbitControls for camera. ACES filmic tone mapping.

**Fire engine & input** (lines ~248-303): A Group of box meshes with a flashing light. WASD/arrows for movement, Space for water spray, R to reset. Movement bounded to [0.02, 0.98].

**Game loop** (lines ~334-400): `animate()` via requestAnimationFrame. `extractPts()` converts the voxel grid to renderable particles each frame. HUD updates every 8 frames. Game ends when burning count reaches 0.

## Key Implementation Details

- Grid indexing: `idx(i,j,k) = i + j*N + k*N*N` — flat array for the 3D grid
- `temp`/`tempN` are double-buffered and swapped each step (line 149)
- Fire particles use a custom ShaderMaterial with vertex sizes and additive blending
- Tree particles only render every other Y layer (`(j&1)===0`) for performance
- Water multiplies temperature by `WATER_STR=0.08` and fuel by 0.92 within radius
