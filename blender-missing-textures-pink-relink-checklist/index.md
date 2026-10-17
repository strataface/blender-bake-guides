---
layout: post
title: "Blender Missing Textures (Pink Materials)? How to Find and Relink Them"
description: "A step-by-step checklist for finding and relinking missing image textures in Blender, with the Blender manual section for each step."
permalink: /blender-missing-textures-pink-relink-checklist/
---

You opened a .blend file after moving the project, copying it to another machine, or getting it from someone else, and some materials no longer show their image textures. Many users describe this as "the textures turned pink." The cause is almost always the same: the file stores a path to each external image, and the image is no longer at that path. This checklist uses Blender's own tools, in order, to confirm which images are missing, relink them, and stop it happening again.

Menu paths follow the Blender 5.2 LTS manual.

## Quick reference

1. **File ‣ External Data ‣ Report Missing Files** to confirm what is missing.
2. **File ‣ External Data ‣ Find Missing Files** and point it at the folder that holds the textures.
3. **File ‣ External Data ‣ Make Paths Relative**, then save.
4. Reload the images (or save and reopen the file).
5. Optional: **Pack Resources** if you need a single self-contained file.

Source: [Topbar ‣ File menu ‣ External Data](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html).

## Why textures go missing

The manual: "External data, like texture images and other resources, can be stored inside the blend-file (packed) or as separate files (unpacked). Blender keeps track of all unpacked resources via a relative or absolute path."

- An **absolute** path (the full path from the system's root) breaks as soon as the image is somewhere else: another drive letter, another user folder, another computer.
- A **relative** path (starting with `//`, the folder of the current blend-file) survives moving the whole project folder, but breaks if the textures are moved separately from the .blend.

Sources: [Topbar ‣ External Data](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html), [Preferences ‣ File Paths](https://docs.blender.org/manual/en/latest/editors/preferences/file_paths.html).

## ☐ 1. Confirm what is missing

Use **File ‣ External Data ‣ Report Missing Files**. The manual: "This option is useful to check if there are links to unpacked files that no longer exist. After selecting this option, a warning message will appear in the Info editor's header. If no warning is shown, there are no missing external files."

Note which images are listed and where they used to be; that tells you which folder to look in.

Source: [Topbar ‣ Report Missing Files](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html).

## ☐ 2. Relink everything from one folder

Use **File ‣ External Data ‣ Find Missing Files**. The manual: "A File Browser will show up. Select the desired directory (or a file within that directory), and a search will be performed in it, recursively in all contained directories. Every missing file found in the search will be recovered."

- Point it at the top-level texture folder; sub-folders are searched too.
- Any image still reported missing afterwards can be fixed by hand (step 4).

Source: [Topbar ‣ Find Missing Files](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html).

## ☐ 3. Make the paths relative and save

The manual warns that Find Missing Files recovers files "as absolute paths, so if you want to have relative paths you will need to select Make Paths Relative." Run **File ‣ External Data ‣ Make Paths Relative** ("Make all paths to external files relative to the current blend-file"), then save.

When you add new images, the **Relative Path** option in the image file browser "Sets the file path to be relative to the currently opened blend-file."

Sources: [Topbar ‣ External Data](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html), [Image Formats ‣ Opening Images](https://docs.blender.org/manual/en/latest/files/media/image_formats.html).

## ☐ 4. Fix single images by hand

For an image that Find Missing Files didn't recover (for example, one that was renamed):

- Select the Image Texture node, or open the image in the Image Editor, and look at the image settings. **Path** is the "Path to the linked file"; **Open** "Opens the File Browser to select a file from a drive."
- Choose the correct file and keep **Relative Path** enabled.

Sources: [Image Settings](https://docs.blender.org/manual/en/latest/editors/image/image_settings.html), [Image Texture Node](https://docs.blender.org/manual/en/latest/render/shader_nodes/textures/image.html).

## ☐ 5. Reload

The manual: "Recovered files might need to be reloaded. You can do that one by one, or you can save the blend-file and reload it again, so that all external files are reloaded at once." For one image, use **Reload** in its image settings ("Reloads the file").

Sources: [Topbar ‣ Find Missing Files note](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html), [Image Settings](https://docs.blender.org/manual/en/latest/editors/image/image_settings.html).

## ☐ 6. UDIM texture sets

A UDIM image is a set of tiles, each its own file. The manual explains that the main tile has index 1001, and that on loading, **Detect UDIMs** "works by detecting if the filename has a .xxxx (four digit number) before the file extension."

- When you relink a UDIM image by hand, keep the tile number pattern in the file name and make sure every tile file is in the same folder.
- If some tiles still look missing, check that the tile files kept their four-digit numbers.

Sources: [UDIMs](https://docs.blender.org/manual/en/latest/modeling/meshes/uv/workflows/udims.html), [Image Formats ‣ Detect UDIMs](https://docs.blender.org/manual/en/latest/files/media/image_formats.html).

## ☐ 7. Textures inside linked libraries

If the material comes from another .blend via **Link**, its images belong to that library file. The manual: "Linked data-blocks are initially not editable," and they are listed in the Outliner's **Blender File** display mode "along with the path of the blend-file they originate from."

- Open the library .blend itself and relink its textures there (steps 1–5), then save it.
- If the library file itself is missing, the manual says Blender "will create placeholder data-blocks to replace missing linked ones," and the data can be restored "by relocating the missing library."

Source: [Link & Append](https://docs.blender.org/manual/en/latest/files/linked_libraries/link_append.html).

## ☐ 8. Prevent it next time: pack or keep paths relative

- **File ‣ External Data ‣ Pack Resources** "Pack[s] all used external files into the blend-file." The manual's Packed Data page: this "allows sharing a full project as a single file." Packing happens on the next save.
- **Automatically Pack Resources** packs files added later too.
- If you'd rather keep textures as separate files, keep them in a folder next to the .blend, use relative paths (step 3), and move the whole project folder together.

Sources: [Packed Data](https://docs.blender.org/manual/en/latest/files/blend/packed_data.html), [Topbar ‣ External Data](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html).

## Symptom table

| Symptom | Check first |
|---|---|
| Textures gone after moving the project | Steps 1, 2, 3 |
| Fine on your machine, missing on another | Step 3 (absolute paths), step 8 |
| One or two images still missing after Find Missing Files | Step 4 |
| Relinked, but the viewport still shows the old state | Step 5 |
| Some UDIM tiles missing | Step 6 |
| Missing textures on linked assets | Step 7 |

## If you relink often

If you regularly relink large projects, [Texture Relink Pro](https://kendeepp.gumroad.com/l/tszcyh?utm_source=seo&utm_campaign=missingtextures) is a Blender add-on for this job. According to its product page, it relinks missing images from a folder you choose (sub-folders optional), matches names that differ only in case or in equivalent extensions such as .jpg/.jpeg and .tif/.tiff (this can be switched off), relinks a UDIM set as one unit, can make images from linked libraries local so the fixed image is the one that renders, and exports a Markdown and JSON report of what changed and what is still missing. New paths are stored relative to the .blend by default. Its listed limits: it does not compare file contents, so two different images with the same name are not told apart; it does not pack or collect textures; it does not repair image sequences or movie clips or fix paths inside the linked library file itself; and it has been tested on Linux with Blender 4.2 LTS and 5.2 LTS (Windows and macOS untested).

## Sources

- Blender Manual, Topbar (File ‣ External Data): https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html
- Blender Manual, Packed Data: https://docs.blender.org/manual/en/latest/files/blend/packed_data.html
- Blender Manual, Image Settings: https://docs.blender.org/manual/en/latest/editors/image/image_settings.html
- Blender Manual, Image Texture Node: https://docs.blender.org/manual/en/latest/render/shader_nodes/textures/image.html
- Blender Manual, Image Formats: https://docs.blender.org/manual/en/latest/files/media/image_formats.html
- Blender Manual, UDIMs: https://docs.blender.org/manual/en/latest/modeling/meshes/uv/workflows/udims.html
- Blender Manual, Link & Append: https://docs.blender.org/manual/en/latest/files/linked_libraries/link_append.html
- Blender Manual, Preferences ‣ File Paths: https://docs.blender.org/manual/en/latest/editors/preferences/file_paths.html

Note: the pink/magenta display of missing images is widely reported by users but isn't described in the Blender manual; menu names may differ in Blender versions other than 5.2 LTS.
