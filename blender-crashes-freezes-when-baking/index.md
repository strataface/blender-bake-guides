---
layout: post
title: "Blender Crashes or Freezes When Baking Textures? A Checklist to Get the Bake Finished"
description: "A step-by-step checklist for Blender texture bakes that crash, freeze or run out of memory, with the Blender manual section for each step."
permalink: /blender-crashes-freezes-when-baking/
---

You click **Bake**, and Blender stops responding, eats all your memory, or closes without warning. You lose the bake, and sometimes the work you hadn't saved. This checklist covers texture baking in Cycles: Base Color, Roughness, Normal, AO and so on. It goes from protecting your work, to finding the usual cause (memory), to running the bake somewhere it can't take your session down with it.

Menu paths follow the Blender 5.2 LTS manual. For bakes that finish but come out black or with seams, see the separate [normal-map bake checklist]({{ "/blender-normal-map-bake-black-seams-checklist/" | relative_url }}).

## Quick reference

1. Save, and check that Auto Save is on, **before** you bake.
2. Watch memory while baking. Running out of memory is the most common cause of crashes.
3. Do a small test bake first: low resolution and low samples.
4. Reduce what the bake has to load: geometry at render level, and huge source textures.
5. If Cycles is set to GPU, try one bake on CPU.
6. For long or risky bakes, run them in a separate background Blender process.
7. After any bake, save the baked image straight away.

## ☐ 1. Protect the work first

The Blender manual lists three safety nets:

- **Save Prompt** asks you to save when you quit.
- **Save Versions** keeps the previous save as `.blend1`.
- **Auto Save** saves a temporary backup "every few minutes (by default, every 2 minutes)".

Check **Preferences ‣ Save & Load ‣ Auto Save** and its **Timer (Minutes)**. After a crash, use **File ‣ Recover ‣ Auto Save** and pick the newest `<filename>_autosave.blend`. Then save it under a new name straight away. The manual warns: "Only one auto-saved file is kept per project."

Sources: [Recovering Data](https://docs.blender.org/manual/en/latest/troubleshooting/recover.html), [Preferences ‣ Save & Load](https://docs.blender.org/manual/en/latest/editors/preferences/save_load.html), [Opening & Saving](https://docs.blender.org/manual/en/latest/files/blend/open_save.html).

## ☐ 2. Is it running out of memory?

The manual's crash page puts **"Running out of memory"** first among the most common causes. To check, keep the system monitor open while the bake runs:

- **Windows:** Task Manager, sorted by Memory.
- **macOS:** Activity Monitor, Memory tab.
- **Linux:** `top -o %MEM`.

If memory climbs to the limit just before Blender freezes or closes, the fixes are in steps 3–5.

The same page also suggests lowering **Preferences ‣ System ‣ Memory & Limits ‣ Undo Steps**. On Linux, it suggests turning off desktop effects that use graphics memory.

Source: [Crashes](https://docs.blender.org/manual/en/latest/troubleshooting/crash.html).

## ☐ 3. Do a cheap test bake first

The manual: "Cycles uses the render settings (samples, bounces, …) for baking." So a bake that seems frozen may simply be running a full-quality render on every pixel of the image.

- Set the target image to **512 × 512** for the test.
- Lower the **Render** samples in Render Properties ‣ Sampling for the test. More samples means more work for each pixel.
- Bake one map at a time.

If the small bake finishes, raise the image size and samples step by step until you find the point where it fails.

Source: [Cycles ‣ Baking](https://docs.blender.org/manual/en/latest/render/cycles/baking.html).

## ☐ 4. Reduce what the bake has to load

- **Subdivision at render level.** The Subdivision Surface modifier has separate **Levels Viewport** and **Render**. A mesh that is light in the viewport can be very heavy at render level. Check both values.
- **Generative modifiers.** A Remesh or similar modifier set very high can multiply the geometry. Apply or lower it before baking.
- **Huge source textures.** Baking many 8K images into one can run out of memory. In one public case, the fix was to downscale the source textures.
- **Very large output images.** 4K and above use much more memory than 1K or 2K. Bake at the size you actually need.

Note: whether the modifiers are evaluated at their Render level during a bake isn't stated on the baking page. Treat render-level settings as part of the bake's load and check them.

Source: [Subdivision Surface Modifier](https://docs.blender.org/manual/en/latest/modeling/modifiers/generate/subdivision_surface.html).

## ☐ 5. GPU or CPU?

If **Render Properties ‣ Device** is set to GPU Compute, try one bake on **CPU**. If the CPU bake finishes and the GPU bake doesn't, look at graphics memory and drivers. The crash page names "issues with graphics hardware or drivers" as the second common cause, and recommends keeping graphics drivers up to date.

Note: graphics cards usually have less memory than the computer, so a bake that fits in system RAM can still fail on the GPU. How much memory a given bake needs isn't documented.

Source: [Crashes](https://docs.blender.org/manual/en/latest/troubleshooting/crash.html).

## ☐ 6. Run the bake in a separate Blender process

A bake started from the open window keeps that Blender busy until it's done, which is a common complaint in forum questions. If it crashes, your whole session goes with it. You can run the bake in a separate process instead, on a copy of the file:

1. Set up the bake as usual: select the object, add an Image Texture node with the target image in each material, and make that node active. Then **save**.
2. Save a copy of the file (**File ‣ Save Copy…**), for example `scene_bake_copy.blend`.
3. Run Blender in background mode on the copy:

```
blender -b scene_bake_copy.blend --python-expr "import bpy; bpy.ops.object.bake(type='AO'); bpy.data.images['Cube_AO'].save(filepath=bpy.path.abspath('//baked/Cube_AO.png'))"
```

Replace `AO` with the bake type you need, and `Cube_AO` with the name of your target image. `-b` runs Blender "in background (often used for UI-less rendering)", and `--python-expr` runs the given Python expression. `bpy.ops.object.bake` bakes "image textures of selected objects". Without the explicit `save`, the result stays inside Blender as an internal image, and Blender prints that the map was "saved to internal image, save it externally or pack it".

If this background process crashes, your open window and its unsaved changes are not affected. Use the system monitor from step 2 to watch the process.

Note: the background process uses the selection, the active Image Texture node and the render settings as they were saved in the copy. If nothing is baked, check those in the copy first.

Sources: [Command Line Arguments](https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html), [bpy.ops.object.bake](https://docs.blender.org/api/current/bpy.ops.object.html), [Image.save](https://docs.blender.org/api/current/bpy.types.Image.html).

## ☐ 7. Save baked images immediately

A bake writes into an image inside Blender. Until you save it (**Image ‣ Save** in the Image Editor) or pack it, a crash loses it. In a long session, save each map as soon as it finishes, before you start the next one.

## ☐ 8. If it still crashes: get the crash log

The manual: "When Blender crashes, it writes out a text file which contains information that may help identify the cause of the crash." On Windows it is named after the file: `test.blend` creates `test.crash.txt`, usually in the Temporary Directory. On macOS, the Crash Reporter window shows the backtrace. For Windows, the crash page also mentions `blender_factory_startup.cmd` in the install folder, which "starts Blender with default settings which is recommended for debugging".

Source: [Crashes](https://docs.blender.org/manual/en/latest/troubleshooting/crash.html).

## Symptom table

| Symptom | Likely cause | Step |
|---|---|---|
| Memory climbs to the limit, then a freeze or crash | Out of memory | 2, 4 |
| "Not responding" for a long time, memory stable | Full render samples at a large size | 3 |
| Crashes only at 4K or above | Output or source images too large | 3, 4 |
| Finishes on CPU, crashes on GPU | GPU memory or drivers | 5 |
| Blender window locked for the whole bake | Bake runs in the UI process | 6 |
| Bake finished but the image is gone after a crash | Baked image never saved | 7 |

## If you bake the same maps for every asset

[ReliaBake](https://kendeepp.gumroad.com/l/srttm?utm_source=seo&utm_medium=referral&utm_campaign=bakecrash) ($19) is a Blender add-on built around steps 6 and 7. According to its product page:

- It runs every map in its own background Blender process from a temporary copy of your .blend, so the file you have open isn't modified.
- Each job has a timeout (automatic: 3× an estimate, at least 60 s, or a value you set). A job that runs over is killed, and the remaining maps keep baking.
- Cancel terminates the worker process, and force-kills it after 3 s if needed.
- After a crash, **Resume Previous Bake** runs only the unfinished maps.
- A crashed worker is retried once with lighter settings, for example AO at 64 → 16 samples.
- Each map is written as `.part.png` first and then renamed, so you don't get half-written files.

It bakes Base Color, Roughness, Metallic, Normal and AO for one mesh object at a time. It has packing presets for glTF, Godot, Unreal and Unity.

Its listed limits:

- No selected-to-active (high-to-low) baking, multi-object batches, Emission/Alpha maps or GPU baking in v0.1.0.
- It has been tested on Linux with Blender 4.2 LTS and 5.2 LTS. Windows and macOS haven't been tested yet.

## Sources

- Blender Manual, Crashes: https://docs.blender.org/manual/en/latest/troubleshooting/crash.html
- Blender Manual, Recovering Data: https://docs.blender.org/manual/en/latest/troubleshooting/recover.html
- Blender Manual, Preferences ‣ Save & Load: https://docs.blender.org/manual/en/latest/editors/preferences/save_load.html
- Blender Manual, Opening & Saving: https://docs.blender.org/manual/en/latest/files/blend/open_save.html
- Blender Manual, Cycles ‣ Baking: https://docs.blender.org/manual/en/latest/render/cycles/baking.html
- Blender Manual, Subdivision Surface Modifier: https://docs.blender.org/manual/en/latest/modeling/modifiers/generate/subdivision_surface.html
- Blender Manual, Command Line Arguments: https://docs.blender.org/manual/en/latest/advanced/command_line/arguments.html
- Blender Python API, bpy.ops.object.bake: https://docs.blender.org/api/current/bpy.ops.object.html
- Blender Python API, Image.save: https://docs.blender.org/api/current/bpy.types.Image.html

Note: menu names may differ in Blender versions other than 5.2 LTS.
