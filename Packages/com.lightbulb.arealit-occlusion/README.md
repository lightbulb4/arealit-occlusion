# AreaLit Occlusion Baker

Safely bake, inspect, adjust, and apply AreaLit occlusion maps with Bakery. Open the tool from **Tools > Lightbulb > AreaLit Occlusion Baker**.

## Requirements

- Unity 2022.3
- A VRChat Worlds 3.x project
- AreaLit for emitter and receiver shaders
- Bakery for scene preparation and baking
- Mochie shaders only when using the Mochie receiver workflow

The package still compiles when Bakery is absent. Baking and scene preparation are disabled with an explanatory warning, while material assignment, output adjustment, and occlusion UV tools remain available.

Generated maps, material variants, and transaction scenes are project-owned assets under `Assets/Lightbulb/AreaLitOcclusion`. Package upgrades never replace that folder.

Open **Tools > Lightbulb > AreaLit Occlusion Baker**.

The editor tool discovers loaded `AreaLit/LightMesh` emitters and materials with an `_AreaLitOcclusion` property. AreaLit emitters are the only user-facing occlusion sources. Normal Unity and Bakery lights are disabled in isolated staging scenes and are never repurposed.

Every automatic checkbox displays its reason:

- Active AreaLit emitters start checked when a safe Bakery proxy can be made. Inactive emitters start unchecked.
- A `MeshRenderer`/`MeshFilter` emitter always receives a transaction-owned proxy built from its exact submesh and transform. Unity's editor-only mesh access supports this even when the imported mesh has Read/Write disabled. Unsupported geometry stays unchecked rather than being approximated.
- Receiver materials appear when AreaLit is enabled, or when their compatible shader has no AreaLit toggle. Materials with AreaLit disabled stay hidden unless **Show materials with AreaLit disabled** is enabled.
- Manual changes are labeled as manually included or excluded.

## Safety model

An occlusion bake never changes lights in the user's scene. All loaded scenes must already be saved and clean. The tool then:

1. Writes a persistent transaction journal under `Library/AreaLitOcclusion`.
2. Saves copies of every loaded scene under this feature's `Transactions` folder.
3. Opens those copies, disables their normal lights, and clones every AreaLit emitter material before recoloring it.
4. Creates transaction-owned Bakery Light Mesh proxies for selected emitters using only the AreaLit emitter geometry, material intensity, and the tool's controlled defaults.
5. Forces Bakery to render into that transaction's isolated `BakeOutput` folder.
6. Restores the original scene setup before applying anything to shared materials.
7. Backs up every existing output texture and material file before changing it.
8. Publishes stable textures under `Generated/<scene GUID>` so scene renames and moves do not change the destination.

Existing texture publication and recovery release Unity's cached asset-file handles before atomically replacing only the image bytes. This keeps the `.meta` GUID stable while avoiding Windows file-sharing failures when the Project window or a loaded material still references the HDR.

If Unity or Bakery stops during the process, the next editor load offers recovery. Staging data is retained rather than deleted automatically. A retained staging scene cannot be used as the source of a new transaction; reopen the original scene first.
Transaction contents are Git-ignored because copied scenes and temporary lightmaps can be very large.

## Proxy behavior

The proxy always uses the AreaLit emitter's exact submesh and transform. Read/Write does not need to be enabled and a hand-authored Bakery mesh is not required. Existing Bakery Light Meshes are ignored. Intensity is derived from the AreaLit material, while conservative Bakery sample, cutoff, self-shadow, and indirect defaults are applied consistently. **Bake intensity** shows that automatic value on every emitter row; editing it creates a manual override, and **Auto** restores the detected value. **Global intensity multiplier** scales every selected emitter after those individual values and defaults to 1. Use **Prepare Scene for Inspection** to review the result before rendering. Skinned meshes, missing geometry, and transforms that cannot be reproduced without shear are blocked instead of guessed.

## Current output mode

The first implementation uses Bakery's color lightmap output and therefore supports red, green, and blue packing. Alpha requires a future shadowmask-based output mode and is blocked rather than silently producing an incorrect map.

Material application changes only `_AreaLitOcclusion`. UV selection, tiling, offset, and other properties remain untouched. A material used across more than one generated lightmap is skipped because choosing one would be ambiguous.

## Output adjustments

**Brightness** and **Contrast** default to 1, which publishes Bakery's original bytes without re-encoding them. Changing either slider updates the thumbnail immediately and applies the same adjustment to the HDR texture during publication. Display compensation is limited to the thumbnail so it follows Unity's normal asset preview instead of appearing artificially dark. Saved Radiance HDR files are adjusted directly in linear RGBE data while preserving their original dimensions and scanline orientation; they do not pass through Unity's imported texture, GPU readback, compression, or display gamma. Empty color channels remain black so contrast changes do not leak between red, green, and blue-packed emitters. The preview automatically uses an occlusion map assigned to a checked receiver, then falls back to a previous or generated map; **Preview override** can show any texture without changing the bake source.

**Save Adjustments to Current Map** applies the sliders immediately to the previewed HDR, EXR, or PNG asset. It creates a persistent recovery copy under `Library/AreaLitOcclusion/ManualAdjustmentBackups`, atomically replaces only the image bytes, preserves the `.meta` GUID and material links, and resets the sliders to neutral to prevent applying the same adjustment twice. **Revert Last Save** restores the original even after a script reload or Unity restart and retains a copy of the adjusted version. Normal bake publication uses the same image path and keeps its transaction backup available for recovery.

## Occlusion UV tools

This section runs independently of baking and considers materials that have AreaLit enabled and an occlusion map assigned. **Match** and **Reset** scan `MeshRenderer` components in every loaded original scene, including disabled components and inactive objects.

- **Match Lightmap Tiling / Offset** copies each renderer's `lightmapScaleOffset` to the AreaLit occlusion texture and selects UV1, matching the previous standalone scanner's intended AreaLit behavior.
- **Reset to Tiling 1,1 / Offset 0,0** restores only the occlusion texture's default scale and offset.

Both actions support Unity Undo and mark changed materials dirty without saving unrelated assets. Shared materials that need conflicting lightmap transforms are identified automatically in the window and remain skipped rather than being overwritten in an arbitrary scan order. Conflict detection and auto-fix exclude inactive GameObjects and disabled `MeshRenderer` components by default; enable **Include disabled objects** to include them. The window reports how many otherwise eligible disabled renderers are hidden. Expand a conflict to inspect its required mappings and select each affected GameObject in the Hierarchy.

**Auto-Fix This Material** and **Auto-Fix All Conflicts** create one generated material variant per distinct required transform, then reassign only the affected renderer material slots. The original material is never modified. Variants live under `Generated/UV Material Variants`, and identical objects that need the same transform continue sharing one variant. Every repair is preflighted against a fresh scene scan, supports Unity Undo for scene assignments, and writes a recovery journal under `Library/AreaLitOcclusion` before creating assets or changing renderers. **Revert Auto-Fix Changes** restores tracked assignments after a restart or interrupted operation and removes generated variants only when no loaded object or saved project asset still references them. Scenes are marked dirty but are never saved automatically. The former `AreaLitMaterialScanner.cs` menu and context-menu tool have been removed.

## Debug and inspection

The **Debug & inspection** section supports two non-destructive checks:

- Assign an existing texture to `_AreaLitOcclusion` on the checked receiver materials. This is a Unity Undo operation and the tool does not auto-save the material assets.
- Prepare the occlusion bake without starting Bakery. The tool creates and opens the same isolated staging scenes used by the normal bake, disables normal lights, applies channel-colored cloned AreaLit materials, creates the Bakery proxies, and pauses for inspection.

While inspection mode is active, use **Bake Prepared Scene** to continue or **Revert to Original Scenes** to leave without baking. A render started directly in Bakery is also detected and tracked. The transaction survives script recompilation as long as its staging scenes remain loaded; otherwise the normal recovery prompt is shown.

The first published texture is converted from Bakery's Lightmap importer type to a regular material texture with sRGB sampling and mipmaps, matching the existing replacement-file workflow. Later bakes preserve the stable texture's `.meta` file and any importer changes made by the artist.
