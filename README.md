# Megascans Plugin for Blender 5.x: unofficial fix

A community-patched copy of Quixel's **Megascans Plugin 3.8** (the Quixel Bridge live link for Blender). With it, clicking **Export** in Bridge imports assets into **Blender 5.0, 5.1 and 5.2** again.

> **Unofficial.** Quixel, Epic Games and the Blender Foundation have nothing to do with this repo. The original plugin is © Quixel AB. Its header comment is kept as is, and the only changes are the compatibility fixes listed below.

## Symptoms this fixes

- **Clicking Export in Bridge does nothing when the render engine is Cycles.** Blender's system console shows:
  `Megascans Plugin Error initializing the import process. Error: 'CyclesRenderSettings' object has no attribute 'feature_set'`
- **OBJ exports never import** (in any engine, from Blender 4.0 on):
  `Calling operator "bpy.ops.import_scene.obj" error, could not be found`
- **Assets with EXR albedo, specular or translucency maps import with a broken material**, also from 4.0 on:
  `enum "Linear" not found`

## Install

Bridge installs the plugin as a Blender *startup script*, not as a regular add-on. To apply the fix:

1. Install the plugin from Bridge once as usual, so the folder exists.
2. Download `MSPlugin-blender5x-fix.zip` from [Releases](../../releases), or just this repo's `MSPlugin/__init__.py`.
3. Replace `__init__.py` in your Blender version's startup folder:
   - **Windows:** `%APPDATA%\Blender Foundation\Blender\<version>\scripts\startup\MSPlugin\__init__.py`
   - **macOS:** `~/Library/Application Support/Blender/<version>/scripts/startup/MSPlugin/__init__.py`
   - **Linux:** `~/.config/blender/<version>/scripts/startup/MSPlugin/__init__.py`
4. Restart Blender.

If you reinstall the plugin from Bridge later, it overwrites this fix. Copy the file back in again.

If you'd rather patch your own copy of the official 3.8 plugin, apply [`blender-5x-fix.patch`](blender-5x-fix.patch) from inside the `startup` folder:

```bash
git apply path/to/blender-5x-fix.patch
```

## What changed

| Problem | Cause | Fix |
|---|---|---|
| Nothing imports with Cycles | `scene.cycles.feature_set` was removed in Blender 5.0 (adaptive subdivision is no longer experimental), and the plugin read it at the start of every import | Read it only if it exists |
| OBJ meshes don't import | The legacy Python OBJ importer (`import_scene.obj`) was removed in Blender 4.0 | Use `wm.obj_import` on 4.0+ with the same options |
| Broken material on EXR color maps | The `"Linear"` color space was renamed to `"Linear Rec.709"` in Blender 4.0 | Use the new name on 4.0+ |
| Cycles adaptive displacement setup fails | The Separate RGB node was removed in 5.0, and `material.cycles.displacement_method` moved to `material.displacement_method` in 4.1 | Use Separate Color, and set the displacement method in the right place for each version |
| Deprecation warnings | `Material.use_nodes` (deprecated in 5.0) and `Thread.getName()` | Skip `use_nodes` on 5.0+, and use `Thread.name` |

Every change is version-gated, so the plugin still behaves the same on Blender 4.x.

### One behaviour difference on 5.x

Before, the plugin wired up a displacement node for Cycles only when the scene's feature set was **Experimental**. That setting no longer exists in Blender 5.x, so the plugin doesn't add displacement there. Assets still import with their normal maps as usual.

## Tested

Tested on Windows 11 with Blender **4.4, 4.5, 5.0 and 5.2.1 LTS**, using synthetic test assets (FBX, OBJ, Alembic, PNG and EXR textures) on both Cycles and EEVEE:

- 9 asset scenarios, each run on both engines: 3D assets, metal and specular workflows, high-poly, plants and billboards, surfaces, snow, EXR albedo, scatter and Alembic. All pass on each version above. The official 3.8 fails the OBJ and EXR cases everywhere, and every case on 5.x with Cycles.
- A full live-link round trip in the Blender 5.2.1 GUI: an asset payload sent over the Bridge socket (port 28888) imported with its materials assigned, on both engines.

Not tested on macOS or Linux, or with every Megascans asset type. If something breaks, please open an issue and include the `Megascans Plugin Error ...` line from Blender's system console.

## Credits

The original Megascans Plugin for Blender is by [Quixel](https://quixel.com/). These are compatibility fixes only. If you represent Quixel or Epic Games and want this repo changed or taken down, please open an issue.
