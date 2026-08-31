# AreaLit Occlusion Baker

AreaLit Occlusion Baker is a Unity editor tool for baking, inspecting, adjusting, and applying AreaLit occlusion maps with Bakery.

It automates temporary bake-scene preparation, restores scene and Bakery state after a bake, creates Bakery light-mesh proxies from AreaLit emitter geometry, applies maps to Mochie materials, synchronizes lightmap tiling and offset, and identifies shared-material UV conflicts.

## Install

Add the Lightbulb VPM listing to VRChat Creator Companion:

`https://lightbulb4.github.io/vpm/index.json`

Then add **AreaLit Occlusion Baker** to a Unity 2022.3 VRChat Worlds project.

## Requirements

- Unity 2022.3
- VRChat Worlds SDK 3.x
- AreaLit materials in the target project
- Bakery for bake and scene-preparation commands
- Mochie shaders only when using Mochie-specific receiver tools

The package still compiles without Bakery. Bakery-dependent commands are disabled with an explanation while material, UV, and image-adjustment tools remain available.

See the [package documentation](Packages/com.lightbulb.arealit-occlusion/README.md) for the workflow and safety model.

## License

[MIT](LICENSE.md)
