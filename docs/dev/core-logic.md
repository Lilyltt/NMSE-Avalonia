# Core Logic Classes

## Overview

Every Logic class in `Core/` is `internal static` -- stateless, no constructors, no instance
fields. They encapsulate game-specific rules, lookups, and data transformations so that UI
panels never contain domain knowledge directly. This separation means logic can be unit-tested
without instantiating any WinForms controls.

**Pattern:**

```
Panel.LoadData(saveData)
  --> Logic.ReadSomething(jsonObject)  // pure extraction
  --> populate UI controls

Panel.SaveData(saveData)
  --> read UI controls
  --> Logic.WriteSomething(jsonObject, values)  // pure mutation
```

Panels call Logic methods with `JsonObject` arguments; Logic methods navigate the JSON tree,
read or write fields, and return typed results. Logic classes never touch UI controls.

---

## Logic Classes

### AccountLogic

| | |
|---|---|
| File | `Core/AccountLogic.cs` |
| Purpose | Manage platform and season reward unlock state across the account file (`accountdata.hg`) and Steam settings MXML |

Loads `accountdata.hg` separately from the main save, extracts `UserSettingsData`, and builds
lists of unlocked season, Twitch, and platform rewards. Provides methods to sync redeemed
reward arrays in `PlayerStateData` and to build display rows by merging database entries with
the unlock set.

**Key methods:**

| Method | Description |
|--------|-------------|
| `LoadAccountData(saveDirectory)` | Loads and parses `accountdata.hg`, returns an `AccountData` bundle |
| `GetUnlockedSet(JsonArray?)` | Converts a JSON array of strings into a case-insensitive `HashSet` |
| `BuildRewardRows(rewardsDb, unlockedSet)` | Merges database entries with the unlock set for display |
| `SaveRewardList(rewards, userSettings, key)` | Writes unlocked IDs back to a JSON array |
| `SyncRedeemedRewards(saveData, seasonRewards, twitchRewards)` | Syncs redeemed arrays in PlayerStateData |

---

### BaseLogic

| | |
|---|---|
| File | `Core/BaseLogic.cs` |
| Purpose | Base building operations -- position swapping and storage container management |

Provides `SwapPositions(a, b)` which exchanges the `Position`, `Up`, and `At` fields between
two base JSON objects. Also defines `ChestInventoryKeys` (10 standard chest names) and
`StorageInventories` -- 8 special storage containers such as Ingredient Storage, Rocket, and
Fishing Platform with their JSON keys, display names, and export filenames.

---

### CompanionLogic

| | |
|---|---|
| File | `Core/CompanionLogic.cs` |
| Purpose | Pet/companion creature operations -- species lookup, deletion, biome types |

Holds a `BiomeTypes` array of 12 biome strings (Lush, Toxic, Scorched, etc.). Provides
`LookupSpeciesName(speciesId)` that queries `CompanionDatabase` for a display name, and
`DeleteCompanion(companion)` that clears all fields (ID, name, seeds, accessories) to reset a
companion slot.

---

### DiscoveryLogic

| | |
|---|---|
| File | `Core/DiscoveryLogic.cs` |
| Purpose | Word/language knowledge, portal glyph tracking, and discovery management |

Defines race columns for 5 playable races (Gek, Vy'keen, Korvax, Atlas, Autophage) with
their index ordinals, and race prefixes that map word group prefixes (`^TRA_`, `^WAR_`, etc.)
to race indices. Also holds a `TechItemTypes` hash set used to filter technology items in
discovery views.

---

### ExosuitLogic

| | |
|---|---|
| File | `Core/ExosuitLogic.cs` |
| Purpose | Constants for exosuit inventory keys and export filenames |

A minimal constants class. Defines cargo and tech inventory JSON keys
(`Inventory`, `Inventory_TechOnly`), export filenames
(`exosuit_cargo_inv.json`, `exosuit_tech_inv.json`), and maximum grid size labels
(`Max Supported: 10x12` for cargo, `10x6` for tech).

---

### FreighterLogic

| | |
|---|---|
| File | `Core/FreighterLogic.cs` |
| Purpose | Freighter class, type, crew, and stat management |

Maps freighter model filenames to type names (Tiny, Small, Normal, Capital, Pirate) and crew
NPC resource paths to race display names (Gek, Vy'keen, Korvax). Provides bidirectional
dictionaries for both lookups. Reads base stat bonuses (`^FREI_HYPERDRIVE`, `^FREI_FLEET`)
from inventory objects via `StatHelper`.

---

### FrigateLogic

| | |
|---|---|
| File | `Core/FrigateLogic.cs` |
| Purpose | Fleet frigate management -- types, grades, races, stats, expeditions |

Defines 10 frigate types (Combat, Exploration, Mining, Diplomacy, Support, Normandy,
DeepSpace, Pirate, GhostShip, etc.), 4 grade levels (C/B/A/S), 3 crew races (mapped
between internal NMS names like "Traders" and display names like "Gek"), and 11 stat
categories. Handles expedition state tracking and level-up calculations.

---

### MainStatsLogic

| | |
|---|---|
| File | `Core/MainStatsLogic.cs` |
| Purpose | Player health, shield, energy, and currency management |

Defines 6 stat fields (Health, Shield, Energy, Units, Nanites, Quicksilver) with their JSON
keys and maximum values. `ReadStatValue` reads from `PlayerStateData`, handling
int/long/double/decimal types and clamping to valid ranges. `WriteStatValues` writes all 6
stats back, using unchecked unsigned int casts for currency fields to support large values.

---

### MilestoneLogic

| | |
|---|---|
| File | `Core/MilestoneLogic.cs` |
| Purpose | Milestone tracking and global statistics management |

Maps milestone section names to icon files (e.g., `UI-MILESTONES.PNG`, `UI-GEK.PNG`,
`UI-PIRATE.PNG`). Locates the `^GLOBAL_STATS` array in `PlayerStateData`, and provides
`ReadStatEntryValue` / `WriteStatEntryValue` helpers that handle both `IntValue` and
`FloatValue` fields in stat entries.

---

### MultitoolLogic

| | |
|---|---|
| File | `Core/MultitoolLogic.cs` |
| Purpose | Multitool class, type, and inventory management |

Defines 14 multitool types (Standard, Rifle, Royal, Alien, Pristine, Sentinel, Switch, Staff,
Atlas, etc.) with their model filenames, plus 4 class grades (C/B/A/S). `BuildToolList`
creates a list of owned multitools from a JSON array, filtering out empty slots by checking
for a valid seed value.

---

### RawJsonLogic

| | |
|---|---|
| File | `Core/RawJsonLogic.cs` |
| Purpose | JSON formatting and parsing utilities for the Raw JSON panel |

Three simple methods: `FormatJson` re-indents a JSON string, `ParseJson` parses raw text
into a `JsonObject`, and `ToDisplayString` converts the in-memory save data to a formatted
display string. Used exclusively by `RawJsonPanel`.

---

### SettlementLogic

| | |
|---|---|
| File | `Core/SettlementLogic.cs` |
| Purpose | Settlement stats, production items, and perk management |

Defines 8 settlement stats (Population, Happiness, Production, Upkeep, Sentinels, Debt,
Alert, Bug Attack) with min/max ranges. Holds a curated list of 50+ allowed production item
names and provides `BuildAllowedProductionItems(database)` to create an ID-to-name mapping
filtered against the `GameItemDatabase`. Caps production quantity at 999 and settlement slots
at 100.

---

### SquadronLogic

| | |
|---|---|
| File | `Core/SquadronLogic.cs` |
| Purpose | Squadron pilot and ship management |

Maps pilot NPC resource filenames to races (including Traveller and Iteration variants) and
ship model filenames to display types (Hauler, Fighter, Explorer, Golden Vector, Boundary
Herald, etc.). Provides bidirectional dictionaries for race lookups (Gek, Vy'keen, Korvax)
and 4 pilot ranks (C/B/A/S).

---

### StarshipLogic

| | |
|---|---|
| File | `Core/StarshipLogic.cs` |
| Purpose | Ship data transformation, type lookups, stat management, and slot operations |

The largest logic class. Maintains a `ShipInfo` dictionary mapping 18+ resource filenames to
display metadata (type name, keywords, cargo/tech max labels). Key methods:

| Method | Description |
|--------|-------------|
| `GetShipInfo(filename)` | Returns display name, cargo label, tech label for a resource path |
| `BuildShipList(shipOwnership)` | Creates a list of owned ships from the JSON array |
| `LoadShipData(ship, playerState, index)` | Extracts all editable ship fields into a typed record |
| `SaveShipData(ship, playerState, values)` | Writes a record of edited values back to the JSON tree |
| `IsCorvette(filename)` | Checks if a ship is a corvette (`BIGGS.SCENE.MBIN`) |
| `GetOwnerTypeForShip(typeName)` | Maps type to inventory owner (Ship, AlienShip, RobotShip, Corvette) |
| `CountValidShips(shipOwnership)` | Counts non-empty slots by checking seed validity |

Uses `StatHelper` to read/write base stats (`^SHIP_DAMAGE`, `^SHIP_SHIELD`, etc.) from
inventory `BaseStatValues` arrays.

---

### ExocraftLogic

| | |
|---|---|
| File | `Core/ExocraftLogic.cs` |
| Purpose | Exocraft management, owner type mapping, and export filename generation |

Defines 6 exocraft vehicle types with their array indices: Roamer (0), Nomad (1), Colossus (2),
Pilgrim (3), Nautilon (5), Minotaur (6). `GetOwnerTypeForVehicle` returns the tech category
owner (Colossus, Submarine, Mech, or Exocraft) used by `InventoryGridPanel` for tech
filtering.

---

## Utility Classes

### ExportConfig

| | |
|---|---|
| File | `Core/ExportConfig.cs` |
| Purpose | User-configurable naming templates and file extensions for inventory import/export |

A public singleton (thread-safe double-checked locking). Stores per-entity file extensions
(e.g., `.nmship`, `.nmtool`) and naming templates with variable substitution:
`${ship_name}_${type}_${class}`. Template variables include `{player_name}`, `{ship_name}`,
`{type}`, `{class}`, `{seed}`, `{race}`, `{rank}`, `{species}`, `{base_name}`, and more.

Serializes to/from JSON via `System.Text.Json`. Provides helper methods to build
`SaveFileDialog` / `OpenFileDialog` filter strings, including multi-format import filters
that support NMSSaveEditor and NomNom wrapper formats alongside native NMSE files.

---

### FileNameHelper

| | |
|---|---|
| File | `Core/FileNameHelper.cs` |
| Purpose | Filesystem-safe filename sanitization |

Single static method: `SanitizeFileName(name)` replaces invalid filename characters and
spaces with underscores, returning `"unnamed"` for null or empty input.

---

### InventoryImportHelper

| | |
|---|---|
| File | `Core/InventoryImportHelper.cs` |
| Purpose | Detect and unwrap inventory JSON from multiple external save editor formats |

When importing an inventory file, the JSON may be wrapped in format-specific envelopes.
This class checks a prioritized list of known wrapper paths:

- `["Store"]` -- NMSSaveEditor multitool format
- `["Data", "Multitool", "Store"]` -- NomNom data envelope
- `["Data", "Vehicle", "Inventory"]` -- NomNom vehicle
- `["Data", "Starship", "Inventory"]` -- NomNom starship
- (and more)

If no known path matches, `FindInventoryBfs` performs a breadth-first search (up to depth 4)
for any object containing a `"Slots"` array. Specialized unwrappers handle NomNom companion,
frigate, and pilot wrappers.

---

### MxmlRewardEditor

| | |
|---|---|
| File | `Core/MxmlRewardEditor.cs` |
| Purpose | Read and write Steam `GCUSERSETTINGSDATA.MXML` for platform rewards |

Auto-detects the Steam install path from the Windows registry and locates the MXML settings
file. Parses nested `<Property>` elements to extract reward IDs (prefixed with `^` to match
database conventions). On write, reconstructs the XML element tree with sequential `_index`
attributes.

---

### SeedHelper

| | |
|---|---|
| File | `Core/SeedHelper.cs` |
| Purpose | Hex seed validation and normalization |

`NormalizeSeed` trims, uppercases hex digits, and ensures a lowercase `0x` prefix (e.g.,
`0xABCD1234`). `IsValidHexSeed` validates format and enforces a maximum length of 18
characters (0x + 16 hex digits).

---

### StatHelper

| | |
|---|---|
| File | `Core/StatHelper.cs` |
| Purpose | Read and write `BaseStatValues` arrays in inventory objects |

Inventory JSON objects contain a `BaseStatValues` array of `{BaseStatID, Value}` entries.
`ReadBaseStatValue(inventory, statId)` searches for a matching `BaseStatID` and returns its
`Value` (or 0.0). `WriteBaseStatValue` updates the value in place. Used by `StarshipLogic`,
`FreighterLogic`, and stat editing panels.
