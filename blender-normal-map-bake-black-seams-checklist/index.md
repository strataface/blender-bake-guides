---
layout: post
title: "Blender Normal Map Bake Black or Showing Seams? A Troubleshooting Checklist"
description: "A step-by-step checklist for Blender normal map bakes that come out black or show seams, with the Blender manual section for each step."
permalink: /blender-normal-map-bake-black-seams-checklist/
---

You baked a normal map in Blender and got one of two results: an image that is completely or partly **black**, or a map that looks fine until you apply it and see **seams** along UV edges. Both problems usually come from a short list of settings. This checklist goes through them in the order that finds the cause fastest, with the relevant Blender manual section for each step.

Menu paths follow the Blender 5.2 LTS manual. They are the same in 4.2 LTS unless noted.

## Quick reference: a correct normal-map bake

Use this as the baseline to compare your file against:

- Render engine: **Cycles** (the Bake panel is under Render Properties ‣ Bake).
- The target mesh has a **UV map** without overlapping islands.
- Every material on the target has an **Image Texture node** pointing at the bake image, **unconnected**, and **selected/active**.
- That image's Color Space is **Non-Color**.
- Bake Type **Normal**, Space **Tangent**.
- High-to-low: high-poly selected first, low-poly **active** (selected last), **Selected to Active** on.
- **Margin** left at its default or higher.
- After baking: **Image ‣ Save As**, because the result lives in the image data-block until you save it.

Sources: [Render Baking](https://docs.blender.org/manual/en/latest/render/cycles/baking.html), [Normal Map Node](https://docs.blender.org/manual/en/latest/render/shader_nodes/displacement/normal_map.html), [Editing Images](https://docs.blender.org/manual/en/latest/editors/image/editing.html).

## Part 1: the bake comes out black

Work through these in order. Most black bakes are fixed by step 1, 2 or 3.

### ☐ 1. Did anything actually bake into this material?

A newly created image is a plain fill. If the bake never wrote to it, you see that fill (often black) and conclude the bake "came out black."

- The manual: "If a material does not contain an active and selected image texture node, nothing will be baked for this material."
- Fix: in **each** material of the target object, add an Image Texture node with the bake image, leave it unconnected, and click it last so it is the active node. Bake again.
- Multi-material objects are the usual trap: one material bakes, the others stay at the fill color.

Source: [Render Baking ‣ Output](https://docs.blender.org/manual/en/latest/render/cycles/baking.html).

### ☐ 2. Is the selection order right? (high-to-low only)

- Selected to Active "bake[s] shading on the surface of selected objects to the active object." The **low-poly must be active**: click the high-poly, then Shift+click the low-poly.
- If the order is reversed, you bake onto the wrong mesh and the image you are looking at doesn't change.

Source: [Render Baking ‣ Selected to Active](https://docs.blender.org/manual/en/latest/render/cycles/baking.html).

### ☐ 3. Do the rays reach the high-poly? (high-to-low only)

The manual: "The rays are cast from the low-poly object inwards towards the high-poly object. If the high-poly object is not entirely involved by the low-poly object, you can tweak the rays start point with Max Ray Distance or Extrusion (depending on whether or not you are using cage)."

Black **areas** (not the whole map) usually mean rays started inside the high-poly or missed it:

- **Cage off:** raise **Extrusion** a little so the rays start just outside the high-poly. Set **Max Ray Distance** so rays don't run into unrelated geometry.
- **Cage on:** raise **Cage Extrusion**, or set a **Cage Object**: a copy of the low-poly inflated with Shrink/Fatten (Alt+S in Edit Mode) until it wraps the high-poly completely. The manual says both meshes "need to have the same Topology (number of faces and face order)."
- Change one value at a time and re-bake. A too-large extrusion causes its own errors because rays catch the wrong surface.

Sources: [Render Baking ‣ Selected to Active / Cage](https://docs.blender.org/manual/en/latest/render/cycles/baking.html), [Shrink/Fatten](https://docs.blender.org/manual/en/latest/modeling/meshes/editing/mesh/transform/shrink-fatten.html).

### ☐ 4. Do the low-poly's normals point outward?

Faces that point inward cast their rays the wrong way. In Edit Mode, select all and use **Mesh ‣ Normals ‣ Recalculate Outside** (Shift+N). The manual: it "flips the orientation of the selected faces where necessary, making them all point outward."

Source: [Mesh Normals](https://docs.blender.org/manual/en/latest/modeling/meshes/editing/mesh/normals.html).

### ☐ 5. Is the image set to Non-Color?

A normal map stores directions, not colors. The Normal Map node docs say the image "should be set to Non-Color mode to give correct results." A normal map viewed or used as sRGB looks wrong and can read much darker or washed out. Set Color Space to **Non-Color** on the Image Texture node (or Image Editor ‣ Sidebar ‣ Image).

Sources: [Normal Map Node](https://docs.blender.org/manual/en/latest/render/shader_nodes/displacement/normal_map.html), [Image Settings ‣ Color Space](https://docs.blender.org/manual/en/latest/editors/image/image_settings.html).

### ☐ 6. Are the UVs valid and non-overlapping?

The manual: "Baking requires a mesh to have a UV map." Faces with no UV area, or islands stacked on top of each other, bake into nothing or into the same pixels, which shows up as black or garbled patches. Unwrap the target again and check it in the UV Editor.

Source: [Render Baking ‣ Setup](https://docs.blender.org/manual/en/latest/render/cycles/baking.html), [UV Unwrapping](https://docs.blender.org/manual/en/latest/modeling/meshes/uv/unwrapping/introduction.html).

### ☐ 7. Not black, but flat light blue?

That is a different problem. A uniform light-blue map (around RGB 0.5, 0.5, 1.0) means "no deviation from the surface." In a single-object bake, nothing is feeding the Principled BSDF's **Normal** input. In a high-to-low bake, **Selected to Active** is probably off.

### ☐ 8. Did the bake finish and get saved?

- Many separate high-poly objects use a lot of memory. The manual: "There is a CPU fixed memory footprint for every object used to bake from. In order to avoid crashes due to lack of memory, the high-poly objects can be joined before the baking process."
- A finished bake is only in memory. Use **Image ‣ Save As** (Shift+Alt+S). If you close Blender first, you reopen the file to an empty image.

Sources: [Render Baking ‣ Selected to Active note](https://docs.blender.org/manual/en/latest/render/cycles/baking.html), [Editing Images ‣ Save As](https://docs.blender.org/manual/en/latest/editors/image/editing.html).

## Part 2: seams in the baked normal map

Seams show as visible lines or lighting jumps along UV island borders or hard edges once the map is applied.

### ☐ 1. Is there enough bake margin?

The manual: "When baking to images, by default a margin is generated around UV 'islands'. This is important to avoid discontinuities at UV seams, due to texture filtering and mip-mapping."

- Keep **Margin** on. Increase **Size** if the seams show at a distance (where mip-maps are used) or the texture will be downscaled.
- **Type:** *Extend* pushes border pixels outwards; *Adjacent Faces* fills the margin "using pixels from adjacent faces across UV seams." Try Adjacent Faces if Extend leaves visible lines.
- Leave enough space between UV islands for the margin to fit.

Source: [Render Baking ‣ Margin](https://docs.blender.org/manual/en/latest/render/cycles/baking.html).

### ☐ 2. Are hard edges split in the UVs?

A hard (sharp) edge needs a UV seam along it. Otherwise one UV island has to store two very different normals across the edge in neighboring pixels, and filtering blends them into a visible line. Mark seams along hard edges and unwrap again. (General baking practice; not stated in the Blender manual.)

### ☐ 3. Does the shading match between bake and use?

- Bake with the **same smooth/flat shading** the mesh will have in the final scene or engine.
- The manual warns about Edge Split during cage baking: "Hard splits (e.g. when the Edge Split Modifier is applied) should be avoided because they will lead to non-smooth normals around the edges."
- Without a cage, rays follow the mesh normals, which the manual says "produces glitches on the edges." If seams sit on hard edges, try baking with a cage.

Source: [Render Baking ‣ Cage](https://docs.blender.org/manual/en/latest/render/cycles/baking.html).

### ☐ 4. Do the space and UV map match when you use the map?

- Normal Map node: "For tangent space normal maps, the UV coordinates for the image must match." Point the Normal Map node at the **same UV map** used for the bake.
- The space in the Normal Map node "should match the setting used for baking" (Tangent in most cases).
- Chain: Image Texture (Non-Color) ‣ Normal Map node ‣ Principled BSDF Normal.

Sources: [Normal Map Node](https://docs.blender.org/manual/en/latest/render/shader_nodes/displacement/normal_map.html), [Render Baking ‣ Normal Space](https://docs.blender.org/manual/en/latest/render/cycles/baking.html).

### ☐ 5. Lighting looks inverted in the game engine?

- Blender "uses the OpenGL convention by default for rendering and baking, where the Y axis in the green channel points up. The DirectX convention ... assumes the Y axis points down."
- Godot requires OpenGL-style maps, according to its StandardMaterial3D docs.
- If the engine expects DirectX-style maps, flip the green channel (in the bake's Swizzle settings or on import in the engine). Not both. The manual lists the bake's channel settings as **Swizzle R, G, B** under Render Properties ‣ Bake ([Render Baking ‣ Normal](https://docs.blender.org/manual/en/latest/render/cycles/baking.html)).

Sources: [Normal Map Node ‣ Convention](https://docs.blender.org/manual/en/latest/render/shader_nodes/displacement/normal_map.html), [Godot StandardMaterial3D](https://docs.godotengine.org/en/stable/tutorials/3d/standard_material_3d.html).

### ☐ 6. Banding that looks like seams?

Stepped bands on smooth curved surfaces are usually 8-bit precision, not UV seams. Bake into a float image and save at higher bit depth. Blender's image formats table lists PNG at 8/16 bit and OpenEXR at 16/32 bit float.

Source: [Image Formats](https://docs.blender.org/manual/en/latest/files/media/image_formats.html).

## Symptom table

| Symptom | Check first |
|---|---|
| Whole map black | Part 1, steps 1, 2, 5 |
| Black patches or spots | Part 1, steps 3, 4, 6 |
| One material fine, another black | Part 1, step 1 |
| Flat light blue | Part 1, step 7 |
| Empty after reopening the file | Part 1, step 8 |
| Lines along UV borders | Part 2, steps 1, 2 |
| Lines along hard edges | Part 2, steps 2, 3 |
| Map looks wrong only when applied | Part 2, step 4 |
| Bumps lit from the wrong side in the engine | Part 2, step 5 |
| Stepped bands | Part 2, step 6 |

## If you bake the same maps over and over

If you also bake Base Color, Roughness, Metallic and AO for the same mesh and pack them for a game engine, [ReliaBake](https://kendeepp.gumroad.com/l/srttm?utm_source=seo&utm_medium=referral&utm_campaign=normalmap) is a Blender add-on that automates those bakes for one mesh object at a time. According to its product page, it checks for missing, overlapping and out-of-range UVs before baking (with one click to select the problem faces), warns when a normal map comes out flat or a map comes out as a single flat color, has an OpenGL/DirectX normal switch, and runs each map in its own background process with a timeout and Resume after a crash. It does not do high-to-low (Selected to Active) baking in version 0.1.0, bakes on the CPU only, and has been tested on Linux with Blender 4.2 LTS and 5.2 LTS (Windows and macOS untested).

## Sources

- Blender Manual, Render Baking (Cycles): https://docs.blender.org/manual/en/latest/render/cycles/baking.html
- Blender Manual, Normal Map Node: https://docs.blender.org/manual/en/latest/render/shader_nodes/displacement/normal_map.html
- Blender Manual, Image Settings: https://docs.blender.org/manual/en/latest/editors/image/image_settings.html
- Blender Manual, Editing Images: https://docs.blender.org/manual/en/latest/editors/image/editing.html
- Blender Manual, Image Formats: https://docs.blender.org/manual/en/latest/files/media/image_formats.html
- Blender Manual, Mesh Normals: https://docs.blender.org/manual/en/latest/modeling/meshes/editing/mesh/normals.html
- Blender Manual, Shrink/Fatten: https://docs.blender.org/manual/en/latest/modeling/meshes/editing/mesh/transform/shrink-fatten.html
- Blender Manual, UV Unwrapping: https://docs.blender.org/manual/en/latest/modeling/meshes/uv/unwrapping/introduction.html
- Godot Docs, StandardMaterial3D: https://docs.godotengine.org/en/stable/tutorials/3d/standard_material_3d.html

Note: putting UV seams on hard edges is common practice but isn't spelled out in the Blender manual; menu names may differ in Blender versions other than 4.2 LTS and 5.2 LTS.
