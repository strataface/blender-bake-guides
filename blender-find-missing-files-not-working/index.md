---
layout: post
title: "Blender \"Find Missing Files\" Not Working? Why Textures Stay Missing and How to Fix Them"
description: "Why Blender's Find Missing Files leaves textures missing (case, extension, duplicate names, linked libraries) and how to fix each case, with manual sources."
permalink: /blender-find-missing-files-not-working/
---

You ran **File ‣ External Data ‣ Find Missing Files**, pointed it at the folder with your textures, and some images are still missing, or a few ended up pointing at the wrong file. Find Missing Files is a simple search, so it fails in a few predictable ways. This checklist goes through each one, with a fix that only uses Blender, plus a short script for the case it can't handle at all: file names that changed.

Menu paths follow the Blender 5.2 LTS manual. If you haven't tried Find Missing Files yet, start with the basic [missing-textures checklist]({{ "/blender-missing-textures-pink-relink-checklist/" | relative_url }}).

## What Find Missing Files actually does

The manual: "A File Browser will show up. Select the desired directory (or a file within that directory), and a search will be performed in it, recursively in all contained directories. Every missing file found in the search will be recovered." It also notes: "Recovered files might need to be reloaded."

The manual doesn't say how a "found" file is decided. In practice, it looks for a file with the **same name** as the missing one. Every failure below comes from that.

Source: [Topbar ‣ File ‣ External Data](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html).

## Quick reference

| What you see | Likely cause | Step |
|---|---|---|
| Still missing, and the file is right there | Name differs in letter case | 2 |
| Still missing after converting PNG/TIFF to JPG | Extension changed | 3 |
| "Found", but the wrong texture shows up | Same file name in several folders | 4 |
| Missing images inside a linked .blend | The path is stored in the library file | 5 |
| Want to switch to another folder (2K/4K), nothing is missing | Find Missing Files skips files that exist | 6 |
| Found, but still pink until you reopen | Images not reloaded | 7 |

## ☐ 1. Check that you picked the right folder

Pick the **top folder** that holds the textures. The search goes into sub-folders. Then run **File ‣ External Data ‣ Report Missing Files** again. The manual: "If no warning is shown, there are no missing external files."

## ☐ 2. Does the name differ only in upper/lower case?

`Metal.png` in the file and `metal.png` on disk can look the same to you, but not to the search. This often happens after files go through Windows, a zip tool or a cloud drive and are then opened on Linux or macOS.

Note: the manual doesn't document whether the name match is case-sensitive. In Blender 5.2 on Linux, a name that differs only in case is not matched. On Windows, and on macOS with its default settings, the file system itself usually ignores case, so the result there may differ.

Fixes:

- Rename the file on disk to match exactly. You can see the name Blender expects in the Image Editor sidebar (**N**) ‣ Image tab, or in the Outliner in **Blender File** mode under Images.
- Or use the script in step 3. It matches names regardless of case.

## ☐ 3. Did the extension change?

You converted a texture library from PNG/TIFF to JPG to save disk space, and now every image is missing. Find Missing Files looks for `wood.jpg` and won't accept `wood.jpeg` or `wood.png`.

Note: this is how it behaves in Blender 5.2. The manual doesn't describe the name matching.

Fix with a short script. Open the **Scripting** workspace, paste this into the Text Editor, set `folder`, save your file first, and click **Run Script**:

```python
import bpy, os
folder = r"/path/to/your/textures"   # the folder that holds your textures
exts = {".png", ".jpg", ".jpeg", ".jpe", ".tif", ".tiff", ".exr", ".tga", ".webp", ".bmp"}

index = {}
for root, _, files in os.walk(folder):
    for f in files:
        stem, ext = os.path.splitext(f)
        if ext.lower() in exts:
            index.setdefault(stem.lower(), []).append(os.path.join(root, f))

for img in bpy.data.images:
    if img.source != 'FILE' or img.packed_file or img.library:
        continue
    path = bpy.path.abspath(img.filepath)
    if os.path.exists(path):
        continue
    stem = os.path.splitext(os.path.basename(path))[0].lower()
    hits = index.get(stem, [])
    if len(hits) == 1:
        img.filepath = bpy.path.relpath(hits[0])
        img.reload()
        print("relinked:", img.name, "->", img.filepath)
    else:
        print("skipped:", img.name, "-", len(hits), "candidates", hits)
```

What it does:

- It only touches images that are **missing** and are single files: it skips UDIM sets, packed images and images from linked libraries.
- It matches on the name **without the extension**, ignoring case.
- It relinks only when there is exactly **one** candidate, and lists the rest, so it never guesses.
- New paths are stored relative to the .blend (`//...`).

Open **Window ‣ Toggle System Console** (Windows), or start Blender from a terminal, to see what it printed. Save the file afterwards.

Note: matching without the extension means `rock.exr` could replace a missing `rock.png` if it's the only candidate. Check the printed list.

Sources: [Image.reload / filepath (Python API)](https://docs.blender.org/api/current/bpy.types.Image.html), [bpy.path](https://docs.blender.org/api/current/bpy.path.html).

## ☐ 4. Is the same file name in more than one folder?

Texture packs often reuse names like `normal.png` or `roughness.jpg` in every material folder. If your search folder holds several files with the same name, Find Missing Files links each missing image to **one** of them. There's no prompt telling you which one it picked.

Note: in Blender 5.2, a name found in two sub-folders was linked to one of them without a warning. The manual doesn't say which one wins.

Fixes:

- Search a narrower folder: the material's own folder, not the whole library.
- Afterwards, check the path of each relinked image in the Image Editor sidebar or the Outliner's **Blender File** mode.
- The script in step 3 skips names with more than one candidate and lists them, so you can relink those by hand.

## ☐ 5. Are the missing images inside a linked library?

If the material comes from a **linked** .blend (a chain icon in the Outliner), its image paths are stored in that library file. The manual: "Linked data-blocks are initially not editable." Two options:

- **Fix it at the source.** Open the library .blend, relink its images there (steps 1–4), and save. Then use **Reload** on the library in the Outliner's **Blender File** mode. If the library file itself moved, use **Relocate**. The manual: right-click a library in Blender File mode "will show the Relocate option".
- **Make it local.** **Object ‣ Relations ‣ Make Local…**, or **Make Local** in the Outliner, turns linked data into a local copy you can edit. You then lose the live link to the library.

Sources: [Link & Append](https://docs.blender.org/manual/en/latest/files/linked_libraries/link_append.html), [Outliner ‣ Editing](https://docs.blender.org/manual/en/latest/editors/outliner/editing.html).

## ☐ 6. Nothing is missing, but you want to switch folders

Find Missing Files only touches files that are missing. To move from a 4K set to a 2K set with the same file names:

- In the Find Missing Files file browser, the operator has a **Find All** option. Its description: "Find all files in the search path (not just missing)". With it enabled, paths are changed for any file with the same name found in that folder.
- To swap a single image everywhere it's used, open the image (Image Editor ‣ Image ‣ Open), then in the Outliner's **Blender File** mode, right-click the old image ‣ **Remap Users**. The manual: it "Replaces all usages of the selected data-block by a different one".

Sources: [bpy.ops.file.find_missing_files](https://docs.blender.org/api/current/bpy.ops.file.html), [Outliner ‣ Editing](https://docs.blender.org/manual/en/latest/editors/outliner/editing.html).

## ☐ 7. Reload, make paths relative, save

- Reload the images, or save and reopen the file. The manual says recovered files "might need to be reloaded".
- Run **File ‣ External Data ‣ Make Paths Relative** so the project survives the next move.
- Save. Run **Report Missing Files** once more to confirm.

Source: [Topbar ‣ File ‣ External Data](https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html).

## If this happens on every project

[Texture Relink Pro](https://kendeepp.gumroad.com/l/tszcyh?utm_source=seo&utm_medium=referral&utm_campaign=findmissing) ($12) is a Blender add-on for exactly these cases. According to its product page:

- It relinks missing images from a folder you choose, with sub-folders optional.
- It finds names that differ only in **case**, and **equivalent extensions** (.jpg/.jpeg/.jpe, .tif/.tiff, then the same name with another image extension). You can switch this off to accept exact names only.
- When the same name exists in several places, it uses the file whose folder layout is closest to the old path, and lists the other candidates in its report.
- It relinks a UDIM set as one unit.
- It can make images from linked libraries local, together with the linked items that use them. This changes your file's data, and can be switched off.
- It exports a Markdown and JSON report of what changed and what is still missing.
- New paths are stored relative to the .blend by default.

Its listed limits:

- It doesn't compare file contents, so two different images with the same name aren't told apart.
- It doesn't pack or collect textures, doesn't repair image sequences or movie clips, and doesn't fix paths inside the linked library file itself.
- It has been tested on Linux with Blender 4.2 LTS and 5.2 LTS. Windows and macOS haven't been tested yet.

## Sources

- Blender Manual, Topbar (File ‣ External Data): https://docs.blender.org/manual/en/latest/interface/window_system/topbar.html
- Blender Manual, Link & Append: https://docs.blender.org/manual/en/latest/files/linked_libraries/link_append.html
- Blender Manual, Outliner ‣ Editing: https://docs.blender.org/manual/en/latest/editors/outliner/editing.html
- Blender Manual, Outliner ‣ Interface (Blender File mode): https://docs.blender.org/manual/en/latest/editors/outliner/interface.html
- Blender Python API, bpy.ops.file.find_missing_files: https://docs.blender.org/api/current/bpy.ops.file.html
- Blender Python API, bpy.types.Image: https://docs.blender.org/api/current/bpy.types.Image.html
- Blender Python API, bpy.path: https://docs.blender.org/api/current/bpy.path.html

Note: menu names may differ in Blender versions other than 5.2 LTS.
