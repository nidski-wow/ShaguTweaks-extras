# ShaguTweaks-extras

This addon extends the capabilities of [ShaguTweaks](https://github.com/shagu/ShaguTweaks) by offering optional modules that can be easily toggled on or off. The new features integrate fully with the "Advanced Options" panel, providing a seamless experience.

For a detailed view of what's new and improved, check out the feature list below.

> [!NOTE]
>
> **This is a fork of [shagu/ShaguTweaks-extras](https://github.com/shagu/ShaguTweaks-extras).**
>
> It tracks upstream and adds no modules of its own. The only file that differs is
> `mods/worldmap-reveal.lua` — see [Fork changes](#fork-changes). Everything else is upstream
> as of `e5140e5`.


## Installation (Vanilla, 1.12)

> [!IMPORTANT]
>
> **This addon requires you to have [ShaguTweaks](https://github.com/shagu/ShaguTweaks) installed.**
>
> Install instructions for ShaguTweaks can be found on the [GitHub Page](https://github.com/shagu/ShaguTweaks).

1. Download **[Latest Version](https://github.com/nidski-wow/ShaguTweaks-extras/archive/master.zip)**
2. Unpack the Zip file
3. Rename the folder "ShaguTweaks-extras-master" to "ShaguTweaks-extras"
4. Copy "ShaguTweaks-extras" into Wow-Directory\Interface\AddOns
5. Restart Wow


## Features

### Action Bar
- **Center Vertical Actionbar**  
  *Center the vertical actionbar on the right side.*

- **Dragonflight Gryphons**  
  *Replaces actionbar gryphons with the dragonflight version.*

- **Floating Actionbar**  
  *Removes all background textures and lets the actionbar float.*

- **Reagent Counter**  
  *Shows a reagent counter on action buttons.*

- **Show Bags**  
  *Shows bag and keyring buttons when using the reduced actionbar layout. Hold Ctrl+Shift to move the bag bar.*

- **Show Micro Menu**  
  *Shows micro menu buttons when using the reduced actionbar layout. Hold Ctrl+Shift to move the micro menu.*

<p align="center"><img src="screenshots/actionbar.gif"></p>

### Chat
- **Chat History**  
  *Save chat history of all non-combatlog windows and restore it on login.*

- **Chat Timestamps**  
  *Add timestamps to chat messages.*

- **Center Text Input Box**  
  *Move the chat input box to the center of the screen.*

- **Enable Text Shadow**  
  *Enable text shadow in all chat frames.*


### General
- **Bag Item Click**  
  *Send items to trade window or auction house search via right click.*

- **Bag Search Bar**  
  *Adds a search field to the bag which allows you to search bag, keyring and bank slots.*

- **Show Energy Ticks**  
  *Show energy and mana ticks on the player unit frame.*

- **Reveal World Map**  
  *Reveals unexplored world map areas and shows exploration hints.*  
  Undiscovered areas are drawn darkened, with a magnifying-glass marker in the middle of each
  (markers can be turned off with the `map.reveal.marker` overwrite). A "Reveal Unexplored"
  checkbox on the world map itself toggles the revealed areas without leaving the map.
  Reworked in this fork — see [Fork changes](#fork-changes).


### Macro
- **Macro Icons**  
  *Detect showtooltip and spells in macros to use them on action buttons.*

- **Macro Tweaks**  
  *Add /equip command to macros, remove #showtooltip from chat and hide macro commands from history.*


### Raid
<img src="screenshots/raid.jpg" float="right" align="right" width="33%">

- **Enable Raid Frames**  
  *Very simple raid frames with only the most basic features.*

- **Hide Party Frames**  
  *Disable default party frames while the raidframes are active.*

- **Show Aggro Indicators**  
  *Show indicators on raid members that are currently attacked by other units. (This only works if the unit is a target of a raid member)*

- **Show Combat Feedback**  
  *Show combat feedback numbers on health bars.*

- **Show Dispel Indicators**  
  *Show indicators for units affected by curse, magic, poison or diseases based on your class.*

- **Show Group Headers**  
  *Display group headers on raid frames*

- **Show Healing Predictions**  
  *Show healing predictions that are received in a healcomm compatible protocol.*

- **Use As Party Frames**  
  *Use raid frames to display party members in regular groups*

- **Use Compact Layout**  
  *Reduces the raid frame size and the displayed elements. As a healer, you should never use this layout.*


## Fork changes

Four commits on top of upstream `e5140e5`, all in `mods/worldmap-reveal.lua`. Every other file,
including the module list and the `.toc`, is untouched upstream code.

The module ships a hardcoded table of world-map overlay geometry (`ShaguTweaks.MapOverlayData`),
because the 1.12 Lua API can only report overlays the player has already discovered. Upstream's
Turtle-branch table was generated from `patch-8.mpq`'s `WorldMapOverlay.dbc`. Clients that resolve
that DBC from a later patch archive get different geometry for every record, so the addon drew the
whole map with stale numbers — mis-sized and mis-placed overlay blocks, in explored zones as well
as unexplored ones.

| Commit | Change |
|---|---|
| [`2a6e247`](https://github.com/nidski-wow/ShaguTweaks-extras/commit/2a6e247) | Regenerated the Turtle overlay table from a live client's effective `WorldMapOverlay.dbc` + `WorldMapArea.dbc`: 53 zones, 663 → 707 entries. DBC texture-name casing preserved, since `alreadyknown[]` compares case-sensitively. |
| [`222c756`](https://github.com/nidski-wow/ShaguTweaks-extras/commit/222c756) | Five latent defects: an argument-shifted `create_hash` call; undiscovered-overlay tint never reset, leaving dimmed textures for the client to reuse; a bare `return` inside the tile loop abandoning the rest of the zone; `unpack_hash` returning a nil name into a tooltip concat; per-update state stashed on the ambient `this`. |
| [`265d9b2`](https://github.com/nidski-wow/ShaguTweaks-extras/commit/265d9b2) | Never redraw overlays the client already drew. The addon now counts the tiles the client consumed and allocates after them, drawing only undiscovered overlays — so explored areas are immune to table staleness. With "Reveal Unexplored" unchecked the addon now draws nothing at all. |
| [`004acf7`](https://github.com/nidski-wow/ShaguTweaks-extras/commit/004acf7) | Draw tinted overlays in the `BORDER` layer instead of `ARTWORK`. Sharing a layer with the client's own overlays made the winner of an overlap arbitrary and unstable between draws, which showed up as straight bright/dark lines at 256-pixel quad boundaries. |

### Known limitation

A minority of overlays have artwork that is opaque flush against its own quad edge. Under any
uniform tint those show a hard straight line where the art was cut — 38 edges across 22 zones,
worst in Blackstone Island and Stonetalon Mountains. This is a property of the source artwork, not
of the tint value, and cannot be fixed by choosing a different one.

### Regenerating the overlay table

The table is client data and goes stale whenever the client's `WorldMapOverlay.dbc` changes, so it
has to be regenerated rather than hand-edited. The procedure: resolve `WorldMapOverlay.dbc` and
`WorldMapArea.dbc` from the highest-priority MPQ in the client's `Data` directory that contains them,
join them on map ID, and emit `TEXTURENAME:width:height:offsetX:offsetY` per overlay, grouped by
`WorldMapArea` map file name. Do not assume patch-archive numbering — enumerate the `Data` directory instead.


## Contact & Support

For anything in the fork changes above, open an issue on
[this repository](https://github.com/nidski-wow/ShaguTweaks-extras/issues). For everything else,
please report upstream at [shagu/ShaguTweaks-extras](https://github.com/shagu/ShaguTweaks-extras).
