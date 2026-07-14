# 🎨 Asset Renamer for Maya

A one-click Maya tool that cleans up your outliner by enforcing a consistent
naming convention across an entire asset hierarchy — groups, subgroups,
geometry, and locators — all from a slick dark blue UI.

No more `pCube1`, `group4`, or `locator2` sitting in a rig or asset file.
Select the top of your hierarchy, hit one button, and everything underneath
gets renamed to match your pipeline's standard.

---

## ✨ What it does

Select the top-level node(s) of an asset in Maya, run the script, and it will
walk **every node beneath your selection** and rename it based on what kind
of node it is:

| Node type                          | Renamed to     | Example              |
|-------------------------------------|----------------|-----------------------|
| Top-level group (your selection)    | `[name]_GRP`   | `robot_GRP`           |
| Any group nested inside             | `[name]_SGRP`  | `armRig_SGRP`         |
| Geometry (mesh transform)           | `[name]_GEO`   | `body_GEO`            |
| Locator                             | `[NAME]_LOC`   | `EYE_LOC`             |

The `name` part is always whatever the node was already called before the
script ran — the tool just figures out the correct suffix and appends it. It
uses the **base name only**, so `[name]` is not literally the word "name."

### 🧠 It's smart about re-runs

If a node already has a suffix from a previous run (in any case — `_grp`,
`_Grp`, `_GRP`, etc.), the script strips it off first before renaming. That
means:

- You can run it as many times as you like without ending up with
  `arm_GRP_GRP_GRP`.
- You can rename something in the outliner, then re-run the tool, and it'll
  just re-apply the correct suffix to your new name.

### 🧩 How it tells a group from a subgroup

- The node(s) **you selected** = groups → `_GRP`
- Any group **nested underneath** your selection, no matter how deep = subgroups → `_SGRP`

### 🔍 How it tells geometry from locators from groups

It looks at the actual **shape node** underneath each transform:

- Has a `mesh` shape → geometry → `_GEO`
- Has a `locator` shape → locator → `_LOC` (and the whole name is uppercased)
- No shape at all → it's just a group/subgroup → `_GRP` or `_SGRP`

---

## 🚀 How to use it

1. Open your Maya scene.
2. In the Outliner or viewport, **select the top-level group(s)** of the
   asset(s) you want to rename. You can select multiple assets at once — the
   tool will process each one independently.
3. Open the **Script Editor** (bottom-right icon in Maya, or `Windows >
   General Editors > Script Editor`).
4. Open a **Python** tab, paste in the entire contents of `asset_renamer.py`,
   and press Enter/Execute — or, if you saved the file to disk, run:

   ```python
   exec(open("/path/to/asset_renamer.py").read())
   ```

5. A small dark blue window titled **"Asset Renamer"** will pop up.
6. Click **"Rename Selected Hierarchy."**
7. A confirmation dialog will let you know it's done. Check your Outliner —
   everything should now follow the naming convention. 🎉

> 💡 **Tip:** Save your scene before running any batch-rename tool for the
> first time on a production file, just in case your hierarchy has something
> unexpected in it (e.g., a mesh with no shape assigned yet).

---

## 🛠️ Making it your own

The script is intentionally kept simple and readable so you can bend it to
your pipeline's exact standards. Here's where to look:

### Change the suffixes

At the top of the file:

```python
SUFFIXES = ['_GRP', '_SGRP', '_GEO', '_LOC']
```

This list is used to **strip old suffixes** before renaming. If you change
your suffix scheme, update this list too, or old names won't be cleaned up
properly.

The actual suffixes get applied inside `rename_hierarchy()`:

```python
if category == 'LOC':
    new_name = base_name.upper() + '_LOC'
else:
    new_name = base_name + '_' + category
```

Want geometry to be `_MESH` instead of `_GEO`? Change the `classify()`
function's return value from `'GEO'` to `'MESH'`, and update this block to
match.

### Change what counts as a "subgroup" vs a "group"

Right now, anything you directly select is a `_GRP`, and anything nested
below it is a `_SGRP`, regardless of depth. If your pipeline only wants
**one level** of subgroup nesting (and deeper groups to still be `_SGRP`, or
maybe `_SSGRP`), you'll want to tweak `classify()`:

```python
def classify(node, is_root):
    ...
    return 'GRP' if is_root else 'SGRP'
```

You could pass a depth counter instead of just `is_root` and branch on that
if you need more granular tiers.

### Change what counts as "geometry"

Currently it only checks for `mesh` shapes:

```python
if shape_type == 'mesh':
    return 'GEO'
```

If you also want NURBS surfaces or curves treated as geometry, add:

```python
if shape_type in ('mesh', 'nurbsSurface', 'nurbsCurve'):
    return 'GEO'
```

### Change locator capitalization behavior

If you don't want locators fully uppercased, just change:

```python
new_name = base_name.upper() + '_LOC'
```

to:

```python
new_name = base_name + '_LOC'
```

### Restyle the UI colors

Look for these two lines in `show_ui()`:

```python
dark_blue = (0.04, 0.07, 0.22)
accent_blue = (0.10, 0.30, 0.62)
```

These are RGB values from `0.0` to `1.0`. Bump them up or down (or plug in
your studio's brand colors) to restyle the window background and the button.

### Skip the confirmation popup

If you're running this on a lot of assets and don't want a dialog every
time, just delete or comment out the `cmds.confirmDialog(...)` block inside
`run_rename()`.

---

## ⚠️ A few things to know before you run it

- The tool renames based on **selection**, so double-check what you have
  selected before clicking the button — it processes the *entire* hierarchy
  under each selected node.
- It does **not** rename shape nodes themselves, only transforms (groups,
  geometry transforms, locator transforms). Maya normally auto-renames
  shapes to match their transform anyway.
- Namespaces are respected — the tool renames the node's local name only and
  won't touch or strip namespaces.
- There's no "undo all" button built in — this is a straightforward
  `cmds.rename()` call, so Maya's native **Undo** (`Ctrl+Z`) will work
  immediately after running it if something looks off.

---

## 📁 Files

- `renamer.py` — the full tool. Contains both the renaming logic and
  the UI, and is fully self-contained (no extra dependencies beyond Maya's
  built-in `maya.cmds`).

Happy renaming! 🧹
