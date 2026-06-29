# ADR 0001: 3D Starfield Rendering Engine

**Date:** 2026-06-29

## Status
Accepted

## Context
Arc requires a highly immersive "3D Starfield" main interface featuring floating platforms, glowing tracks, deep-space particle effects, and cinematic camera zooms. The app must run smoothly on both Windows and Android. Implementing true 3D using GLTF/GLB models is overkill since the core entities are procedural nodes and connections rather than rigged 3D models. However, raw Flutter Canvas operations would require complex custom matrix math for camera logic and particle systems.

## Decision
We will use the **Flame Engine** combined with standard **Flutter UI Overlays**.

Specifically:
1. **Engine Layer:** Flame will handle the core 2.5D rendering (Starfield View), including the Parallax background, particle systems for glowing tracks, and the CameraComponent for zooming/panning.
2. **UI Layer:** All interactive panels, galleries, and glassmorphic dialogs will be pure Flutter Widgets overlaid on top of the Flame widget.
3. **Performance Strategy:** 
   - Flame's built-in culling will be used.
   - Nodes will be loaded in temporal Chunks (e.g., ±3 months).
   - Distant nodes will aggregate into Clusters.
   - Thumbnails will be rendered as low-res textures in Flame.
4. **Fallback Plan:** Phase 1 requires a prototype validation. If Flame causes >20MB bundle size bloat or compatibility/performance issues on either platform, we will fallback to a pure Flutter Canvas 2.5D projection approach.

## Consequences
- **Positive:** Drastically reduces the math required for camera manipulation and particles. Enables rapid prototyping of the visual effects. Maintains native Flutter performance for data-heavy UI like galleries.
- **Negative:** Introduces a dependency on a game engine for a utility app, which could affect startup time or bundle size (hence the prototype constraint). Requires managing the state bridge between Flame components and Flutter widgets.