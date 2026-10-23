# More Upgrade Materials

Loot gives double Chroma Catalysts, Colours of Lumina and Recoats; Petanks give three times as many.

A mod for Clair Obscur: Expedition 33 (Unreal Engine 5.4), made with game version `1.5.6.0`. This README is
for developers. What the mod does, how to install it and its compatibility are in [openmods.md](openmods.md), the mod
page on [openmods](https://openmods.nexus).

## How it works

The game keeps the loot of upgrade items in five data tables in `/Game/Gameplay/Inventory/LootSystem/Content/`:

| Table | Used by |
|---|---|
| `DT_LootTable_UpgradeItems_Regular` | the enemy archetypes Weak, Regular, Strong, Elusive |
| `DT_LootTable_UpgradeItems_Elite` | Elite and the bosses (Boss, Boss_NoAchievement, HardOnly_Boss) |
| `DT_LootTable_UpgradeItems_Alpha` | Alpha |
| `DT_LootTable_UpgradeItems_Exploration` | the chests (`DT_ChestsContent`) |
| `DT_LootTable_UpgradeItems_Petank` | Petank |

Each row is a section with a level range, a `ChancePercent` and loot entries (item, `Weight`, `Quantity`).
`BP_LootSystemComponent` keeps the sections whose range holds the encounter level, lets each drop with its chance,
picks one entry by weight and gives its `Quantity`. The mod changes only the 39 quantities: x2 in the first four
tables, x3 in the Petank table. Items: `UpgradeMaterial_Level1` to `5` (Chroma Catalyst to Perfect Chroma Catalyst),
`Consumable_LuminaPoint` (Colour of Lumina), `Consumable_Respec` (Recoat).

The tables' structs `S_LootTableSection` and `S_LootTableEntry` were made in the editor, so they are not in the
mapping file: `build.ps1` reads them from the installed game (`structs` in `mod.json`).

The mod replaces whole assets. A mod that changes the same assets conflicts with this one, and a game update that
changes them needs a new build of the mod.

## Build

Requirements:

- Windows PowerShell 5.1 (part of Windows).
- [UAssetGUI](https://github.com/atenfyr/UAssetGUI/releases) v1.1.0 and [retoc](https://github.com/trumank/retoc/releases)
  v0.1.5. `build.ps1` looks for `_tools\uassetgui\UAssetGUI.exe` and `_tools\retoc\retoc.exe` in the parent folders
  of the project; set `UE5_TOOLS` to a folder that holds both to use another place.
- The game's mapping file, `Expedition33Mappings-1.5.6.usmap`, in `%LOCALAPPDATA%\UAssetGUI\Mappings\`.
- The game's own structs that the assets use (`structs` in `mod.json`; structs made in the editor are not in the
  mapping file): from `deps\` when it holds them, else from the installed game. They are game files and stay out of
  this repository.
- For the check of the build (optional): Python 3, the game installed on the same drive as the project, and, when
  the game's files are encrypted, its AES key in `_tools\gamedata\expedition33\game.json` (`{"aesKey": "0x..."}`).

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1             # dist\expedition33_moreupgradematerials_P.pak/.utoc/.ucas
powershell -ExecutionPolicy Bypass -File .\build.ps1 -Deploy     # build, then copy the files to Paks\~mods
powershell -ExecutionPolicy Bypass -File .\deploy.ps1 -Wait      # copy the files when the game is closed
```

The build converts every asset source in `src` (UAssetGUI JSON) into a legacy `.uasset`/`.uexp` (UAssetGUI), packs
them into an IoStore container (retoc), and then checks the result: it converts the container back and compares
each asset with its source (`scripts\check_build.py`).

`deploy.ps1` copies the three files to `<game folder>\Sandfall\Content\Paks\~mods`. The game folder is
`$env:UE5_GAMES_DIR\Clair Obscur Expedition 33`; `-GameDir` or `-ModsDir` override it. The running game locks the files, so the
script stops (exit code 2) or, with `-Wait`, waits until the game is closed.

Versioning: every change raises the second number in `mod.json` (1.0 -> 1.1): `python scripts\bump_version.py`.

## Project layout

```
src/                               Mod source: the changed assets as UAssetGUI JSON, in the game's folder layout
dist/                              Build output: the .pak/.utoc/.ucas for Paks\~mods (git-ignored)
docs/                              Cover and screenshots for openmods.md
scripts/check_build.py             Compares the built assets with their sources
scripts/bump_version.py            Raises the version in mod.json
build.ps1                          Builds dist\ from src\ and checks the result
deploy.ps1                         Copies the build to the game's Paks\~mods folder
mod.json                           Name, version, game, engine and file names
```

The first commit with an asset holds the original asset, so `git diff` against it shows exactly what the mod
changes.

## License

MIT, see [LICENSE](LICENSE).
