# UI Layer

## Overview

The UI layer (`UI/`) is a WinForms application built around a tabbed `MainForm` that hosts
specialized panels. Each panel is a `UserControl` that follows a consistent
`LoadData` / `SaveData` contract. Panels delegate all game-specific knowledge to
[Core Logic classes](core-logic.md) and use [Data databases](data-layer.md) for item
lookups and icon rendering.

**Key design principles:**

1. **Deferred loading** -- only the active tab is populated on file open; other tabs load on
   first selection. This keeps startup fast.
2. **Direct mutation** -- inventory edits write directly to the in-memory `JsonObject` slots
   during interaction, not on save. `SaveData` is mostly a sync pass for non-inventory
   fields.
3. **Flicker-free updates** -- `RedrawHelper.Suspend/Resume` suppresses `WM_PAINT` during
   bulk control creation; `SuspendLayout/ResumeLayout` prevents intermediate layout passes.

---

## MainForm

| | |
|---|---|
| File | `UI/MainForm.cs` |
| Purpose | Top-level window that orchestrates all panels, handles file open/save, and manages deferred tab loading |

### Tab Layout

MainForm uses a `TabControl` with 15 tabs:

| Index | Panel | Description |
|-------|-------|-------------|
| 0 | MainStatsPanel | Player health, currency, game mode |
| 1 | ExosuitPanel | Personal cargo and tech inventories |
| 2 | MultitoolPanel | Multitool selection and editing |
| 3 | StarshipPanel | Ship selection and editing |
| 4 | FleetPanel | Container for Freighter, Frigate, Squadron sub-tabs |
| 5 | ExocraftPanel | Vehicle inventories |
| 6 | CompanionPanel | Pet management |
| 7 | BasePanel | Base building and storage containers |
| 8 | DiscoveryPanel | Words, glyphs, recipes (includes RecipePanel as sub-tab) |
| 9 | MilestonePanel | Milestones and global statistics |
| 10 | SettlementPanel | Settlement stats, production, perks |
| 11 | ByteBeatPanel | ByteBeat music editor |
| 12 | AccountPanel | Platform/season/Twitch rewards |
| 13 | ExportConfigPanel | Export naming and extension preferences |
| 14 | RawJsonPanel | Raw JSON tree viewer/editor |

### Initialization Flow

```
1. LoadDatabase()
   - Load GameItemDatabase from Resources/db/
   - Load RecipeDatabase, RewardDatabase, etc.
   - Set JsonNameMapper globally on JsonParser
   - Initialize LocalisationService with lang/ directory
   - Call SetDatabase() and SetIconManager() on all panels
   - Start icon preload in background (form Opacity = 0)

2. Form.Shown event
   - Await icon preload task
   - Set Opacity = 1 (reveal form)

3. User opens a save file
   - SaveFileManager.LoadSaveFile() -> JsonObject
   - RegisterContextTransforms()
   - Load tab 0 (MainStatsPanel) immediately
   - Other tabs load on first selection
```

### Menu Structure

| Menu | Position | Items |
|------|----------|-------|
| **File** | 1 | Open Save Directory, Load Save File, Save, Save As, Exit |
| **Edit** | 2 | Reload, Restore Backup (All), Restore Backup (Single) |
| **Tools** | 3 | Export JSON, Import JSON |
| **Language** | 4 | All 16 BCP 47 language tags (en-GB, fr-FR, it-IT, de-DE, es-ES, ru-RU, pl-PL, nl-NL, pt-PT, es-419, pt-BR, zh-CN, zh-TW, ko-KR, ja-JP, en-US) |
| **Help** | 5 | GitHub Page, Sponsor Development, About |

The **Language** menu allows switching the display language for item names, descriptions, and
subtitles. Each tag corresponds to a localisation JSON file in `Resources/json/lang/`. When
a language is selected:

1. `LocalisationService.LoadLanguage(tag)` loads the flat key-value JSON file.
2. `GameItemDatabase.ApplyLocalisation()` swaps all item display fields (Name, NameLower,
   Subtitle, Description) with localised values, backing up English originals. Uses
   DescriptionLocStr-based fallback when the primary Name_LocStr chain fails (common for
   procedural technology items whose Name_LocStr base key differs from the lang-file base).
3. `RewardDatabase.ApplyLocalisation()` swaps reward display names.
4. `AccountPanel.RefreshRewardNames()` re-resolves cached reward display strings from the
   now-localised GameItemDatabase and RewardEntry data.
5. `WordDatabase.ApplyLocalisation()` swaps word display text (with group key fallback).
6. All other databases (Recipe, Title, FrigateTrait, SettlementPerk, WikiGuide) apply
   localisation similarly.
7. `FrigatePanel.RefreshTraitCombos()` and `SettlementPanel.RefreshPerkCombos()` repopulate
   combo boxes with newly localised trait/perk display names.
8. The language preference is persisted via `AppConfig.Language` for next startup.
9. The status bar shows the language tag and count of localised items/rewards/words.

The app defaults to **en-GB** on first launch. On subsequent launches it restores the last
selected language from `AppConfig` via `ApplyStartupLanguage()` (called after `LoadDatabase()`).

If the file is not found, all databases are reverted to English defaults and the status bar
shows a fallback message. Selecting a different language re-applies from the English
baseline, so switching languages does not accumulate drift. All internal editor logic
(string comparisons, filtering) always runs against IDs, not display names.

### Deferred Tab Loading

```
OnTabChanged:
  if tab not in _loadedTabIndices:
    Hide panel -> SuspendLayout -> LoadData() -> ResumeLayout -> Show panel
    Add to _loadedTabIndices
  
  if switching to RawJsonPanel:
    SyncAllPanelData() -> RefreshTree()
```

Only panels that have been loaded are synced during `SyncAllPanelData`. This avoids
calling `SaveData` on panels that were never shown.

### Data Flow

```
Open File:
  SaveFileManager.LoadSaveFile(path)
    -> JsonObject (root)
    -> RegisterContextTransforms(root)
    -> _currentSaveData = root

Per Panel:
  panel.LoadData(_currentSaveData)
    -> panel extracts its slice via Logic constants
    -> populates UI controls

User Edits:
  InventoryGridPanel writes directly to JsonObject slots
  Other controls raise DataModified -> _hasUnsavedChanges = true

Save File:
  SyncAllPanelData()
    -> each loaded panel.SaveData(_currentSaveData)
  SaveFileManager.SaveToFile(path, _currentSaveData, ...)
```

---

## Panel Architecture

Every panel follows this contract:

```csharp
public partial class XxxPanel : UserControl
{
    public void SetDatabase(GameItemDatabase? database);
    public void SetIconManager(IconManager? iconManager);
    public void LoadData(JsonObject saveData);
    public void SaveData(JsonObject saveData);
    public event EventHandler? DataModified;
}
```

Panels extract their relevant JSON subtree from `saveData` (usually via
`saveData.GetObject("PlayerStateData")`) and forward it to child controls or use Logic
class methods to read/write fields.

### Panel Reference

#### Equipment Panels

| Panel | File | Description |
|-------|------|-------------|
| **ExosuitPanel** | `UI/Panels/ExosuitPanel.cs` | Two `InventoryGridPanel` instances for cargo (10x12 max) and tech (10x6 max) inventories. Uses `ExosuitLogic` for JSON key constants and export filenames. |
| **MultitoolPanel** | `UI/Panels/MultitoolPanel.cs` | Selector for up to 6 multitools. Displays name, seed, type, class. Each tool has a tech inventory grid. Uses `MultitoolLogic` for type lookups and `SeedHelper` for seed validation. |

#### Equipment Panel

| **StarshipPanel** | `UI/Panels/StarshipPanel.cs` | Selector for up to 12 ships. Displays name, seed, type, class, and base stats (damage, shield, hyperdrive, maneuverability). Three inventory grids (cargo, tech, cargo overflow). Uses `StarshipLogic` extensively for type info, stat reading, and corvette detection. |

#### Fleet Panels

| Panel | File | Description |
|-------|------|-------------|
| **FleetPanel** | `UI/Panels/FleetPanel.cs` | Container panel with sub-tabs for Freighter, Frigate, and Squadron. Manages shared `PlayerStateData` reference. |
| **FreighterPanel** | `UI/Panels/FreighterPanel.cs` | Freighter type, class, crew race, and base stats. Cargo and tech inventory grids. Uses `FreighterLogic` for type/race mapping. |
| **FrigatePanel** | `UI/Panels/FrigatePanel.cs` | List of up to 30 frigates with type, grade, race, and 11 stat categories. Uses `FrigateLogic` for type/grade lookups and `FrigateTraitDatabase` for trait editing. `RefreshTraitCombos()` repopulates trait dropdowns after JSON databases load and after language changes. |
| **SquadronPanel** | `UI/Panels/SquadronPanel.cs` | Squadron pilot management: ship type, pilot race, rank. Uses `SquadronLogic` for resource-to-type mapping. |

#### Vehicle / Companion Panels

| Panel | File | Description |
|-------|------|-------------|
| **ExocraftPanel** | `UI/Panels/ExocraftPanel.cs` | Tabs for 6 vehicle types (Roamer, Nomad, Colossus, Pilgrim, Nautilon, Minotaur). Each has cargo and tech inventory grids. Uses `ExocraftLogic` for owner type mapping. |
| **CompanionPanel** | `UI/Panels/CompanionPanel.cs` | List of up to 18 companion creatures. Species selection from `CompanionDatabase`, biome type, seed editing, appearance customization via `CreaturePartDatabase`. Uses `CompanionLogic`. |

#### Structure Panels

| Panel | File | Description |
|-------|------|-------------|
| **BasePanel** | `UI/Panels/BasePanel.cs` | Base building management: position editing, 10 standard chest inventories, 8 special storage containers. Uses `BaseLogic` for position swapping and storage key definitions. |
| **SettlementPanel** | `UI/Panels/SettlementPanel.cs` | 8 settlement stats with min/max sliders, production item selection, perk list with beneficial/detrimental indicators. Uses `SettlementLogic` and `SettlementPerkDatabase`. `RefreshPerkCombos()` repopulates perk dropdowns after JSON databases load and after language changes. |

#### Discovery / Knowledge Panels

| Panel | File | Description |
|-------|------|-------------|
| **DiscoveryPanel** | `UI/Panels/DiscoveryPanel.cs` | Word/language knowledge grid (5 races), portal glyph tracking with `CoordinateHelper` glyph rendering, galaxy info. Hosts `RecipePanel` as a sub-tab. Uses `DiscoveryLogic` and `WordDatabase`. |
| **RecipePanel** | `UI/Panels/RecipePanel.cs` | Displays crafting and refining recipes from `RecipeDatabase`. Bidirectional lookup: what produces an item, what uses it as an ingredient. `RefreshLanguage()` repopulates the grid with localised item names on language change. |
| **MilestonePanel** | `UI/Panels/MilestonePanel.cs` | Milestone progress and global stat editing. Section icons from `MilestoneLogic.SectionIconMap`. Read/write via `MilestoneLogic` stat helpers. |

#### Gameplay Panels

| Panel | File | Description |
|-------|------|-------------|
| **MainStatsPanel** | `UI/Panels/MainStatsPanel.cs` | Player name, health, shield, energy, units, nanites, quicksilver. Game mode display, play time, save metadata. Includes a Guides sub-tab with wiki topic grids (Icon + categories from `WikiGuideDatabase`, using English categories for grid placement via `GetEnglishCategory()`) and a Titles sub-tab. Uses `MainStatsLogic` for stat field definitions and read/write. |
| **ByteBeatPanel** | `UI/Panels/ByteBeatPanel.cs` | ByteBeat music composition editor. Manages the ByteBeat JSON subtree in PlayerStateData. |
| **AccountPanel** | `UI/Panels/AccountPanel.cs` | Manages rewards from `accountdata.hg`: season rewards, Twitch rewards, platform rewards. Uses `AccountLogic` for data loading and `MxmlRewardEditor` for Steam MXML sync. `RefreshRewardNames()` re-resolves cached display strings after a language change so reward grids show localised names. |

#### Configuration / Debug Panels

| Panel | File | Description |
|-------|------|-------------|
| **ExportConfigPanel** | `UI/Panels/ExportConfigPanel.cs` | UI for `ExportConfig` singleton: file extension preferences, naming templates with variable substitution. No `SaveData` method (config is saved separately). |
| **RawJsonPanel** | `UI/Panels/RawJsonPanel.cs` | TreeView-based JSON editor. Displays the full save tree with expandable nodes. `RefreshTree()` rebuilds from `_currentSaveData` after all panels sync. Uses `RawJsonLogic` for formatting. |

---

## Reusable Controls

### InventoryGridPanel

| | |
|---|---|
| File | `UI/Controls/InventoryGridPanel.cs` |
| Purpose | Visual inventory grid with item management, used by nearly every panel |

The most complex UI component. Renders inventory slots as a 10-column grid with 72x104 px
cells. Each cell shows an icon, item count, charge indicator, and supercharge glow.

**Configuration methods** (called before `LoadInventory`):

| Method | Description |
|--------|-------------|
| `SetDatabase(database)` | Set item database for picker and display |
| `SetIconManager(iconManager)` | Set icon source for cell rendering |
| `SetInventoryOwnerType(type)` | "Suit", "Ship", "Weapon", "Freighter", "Vehicle" -- filters tech items |
| `SetIsTechInventory(bool)` | Show charge indicators |
| `SetIsCargoInventory(bool)` | Reject technology items |
| `SetSuperchargeConstraints(maxSlots, maxRow)` | Limit supercharged slot placement |
| `SetCorvetteContext(saveRoot, shipIndex)` | Enable corvette base part resolution |
| `SetExportFileName(name)` | Default filename for export dialogs |
| `SetExportFileFilter(export, import, ext)` | File dialog filters |

**Item editing flow:**

1. User right-clicks a slot -> context menu (Add / Remove / Copy / Paste / Export / Import)
2. "Add" opens `ItemPickerDialog` with cascading filters: Type -> Category -> Item
3. Selected item is written directly to the slot's `JsonObject`
4. Grid refreshes the affected cell

**Tech adjacency rendering:** When `SetIsTechInventory(true)`, the grid queries
`TechAdjacencyDatabase` to find items with matching `BaseStatType` values. Adjacent tech
items receive colored synergy borders using the `LinkColourHex` from the database.

**Corvette base part resolution:** For corvette ships, `CV_` technology items map to
buildable base parts. `SetCorvetteContext` collects base part objects; the grid uses
`GameItemDatabase.CorvetteBasePartTechMap` and greedy first-match to resolve display names.

---

### ItemPickerDialog

| | |
|---|---|
| File | `UI/Controls/ItemPickerDialog.cs` |
| Purpose | Modal dialog for selecting a game item with cascading category filters |

Presents three cascading filter lists: inventory type, category, and individual items.
Supports keyboard navigation (arrow keys + Enter). Used by `InventoryGridPanel` when adding
or replacing items.

---

## UI Utilities

### FontManager

| | |
|---|---|
| File | `UI/Util/FontManager.cs` |
| Purpose | Manage embedded NMS-style GeoSans font via `PrivateFontCollection` |

Loads a custom font from embedded resources. Provides `GetFont(size, style)` to create font
instances without requiring the font to be installed system-wide.
The font is provided by NMSCD's No Man's Sky Universal Font via the OFL license.

---

### RedrawHelper

| | |
|---|---|
| File | `UI/Util/RedrawHelper.cs` |
| Purpose | Suppress and resume control painting to eliminate flicker |

Sends `WM_SETREDRAW` messages to controls:

```
RedrawHelper.Suspend(control)  // WM_SETREDRAW = FALSE
// ... rebuild child controls ...
RedrawHelper.Resume(control)   // WM_SETREDRAW = TRUE, Invalidate, Update
```

Used by `InventoryGridPanel` during grid reconstruction to prevent visible intermediate
states.

---

### ColorEmojiLabel

| | |
|---|---|
| File | `UI/Controls/ColorEmojiLabel.cs` |
| Purpose | Label control that renders color emoji/glyphs via GDI+ on Windows 10+ |

Standard WinForms labels render emoji in monochrome. This custom control uses GDI+
`DrawString` with the Segoe UI Emoji font to produce full-color emoji rendering. Falls back
to standard rendering on older Windows versions. Used minimally for symbol rendering.
