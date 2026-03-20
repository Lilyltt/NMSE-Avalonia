# Player and World Features Comparison

## Player Stats

| Stat | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Health** | ✅ (int via ReadStatValue) | ✅ (has copy-paste bug in setter) | ✅ |
| **Shield** | ✅ | ✅ | ✅ |
| **Energy** | ✅ | ✅ | ✅ |
| **Units** | ✅ (unsigned-in-signed conversion) | ✅ (GetCurrency/SetCurrency) | ✅ |
| **Nanites** | ✅ (same conversion) | ✅ | ✅ |
| **Quicksilver (Specials)** | ✅ | ✅ | 🟡 |
| **Save Name** | ✅ | ✅ (Waypoint+ gated) | 🟡 |
| **Save Summary** | ✅ | ✅ (Waypoint+ gated) | 🟡 |
| **Third Person Camera** | ✅ | ✅ | ❌ |
| **Player State (LastKnownPlayerState)** | ✅ | ✅ (via libNOM.io) | ❌ |
| **Portal Interference** | ✅ | ✅ (via libNOM.io) | ❌ |
| **Total Play Time** | ✅ | ✅ | ❌ |
| **Game Mode Selector** | ✅ | ✅ | ❌ |
| **Health Max** | 🟡 (game clamps) | ✅ (computed) | ❌ |

### Currency Storage Note

NMS stores currency values (Units, Nanites, Quicksilver) as signed 32-bit integers where values above `int.MaxValue` wrap to negative. Both NMSE and NomNom correctly handle the unsigned-to-signed conversion for display and save-back.

## Difficulty Presets

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Current preset** | ✅ (DifficultyState.Preset) | ✅ (via PresetsEnum) | 🟡 (partial read) |
| **Easiest used preset** | ✅ | ✅ | ❌ |
| **Hardest used preset** | ✅ | ✅ | ❌ |
| **Preset values** | ✅ Invalid, Custom, Normal, Creative, Relaxed, Survival, Permadeath | ✅ Same via PresetsEnum | ❌ |

## Coordinate / Location Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Galaxy display** | ✅ | ✅ | ❌ |
| **Glyph display** | ✅ | ✅ | ❌ |
| **Coordinate display** | ✅ | ✅ | ❌ |
| **Editable coordinates** | ✅ (Galaxy, VoxelX/Y/Z, SolarSystem, Planet NUDs) | ✅ (via ViewModel) | ❌ |
| **Galaxy database** | ✅ (GalaxyDatabase with types + colours) | ✅ | ❌ |

## Title Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Title selection** | ✅ (Titles subpanel in MainStats) | ✅ | ❌ |
| **Title database** | ✅ (TitleDatabase from extractor) | ✅ | ❌ |

## Companion / Pet Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Companion name (CustomName)** | ✅ | ✅ | 🟡 |
| **Custom species name** | ✅ | ✅ | ❌ |
| **Species (CreatureID) dropdown** | ✅ (69 species from CompanionDatabase) | ✅ | ❌ |
| **Creature type (60 types)** | ✅ (CompanionDatabase.CreatureTypes) | ✅ | 🟡 |
| **6 seed fields** | ✅ (validated hex) | ✅ | 🟡 |
| **Scale** | ✅ | ✅ | ❌ |
| **Predator toggle** | ✅ | ✅ | ✅ |
| **Trust / Helpfulness** | ✅ | ✅ | ❌ |
| **BirthTime** | ✅ (DateTimePicker, Unix timestamp) | ✅ (DateTime) | ❌ |
| **LastEggTime** | ✅ (DateTimePicker, Unix timestamp) | ✅ (DateTime) | ❌ |
| **IsUnlocked (per-slot)** | ✅ (UnlockedPetSlots toggle) | ✅ | ❌ |
| **Creature Part Descriptors** | ✅ (CreaturePartDatabase, 55 types, 3056 nodes) | ❌ | ❌ |
| **Create from empty slot** | ✅ (ActivateSlotIfEmpty) | ✅ | ❌ |
| **Delete / clear companion** | ✅ (UA reset to 1111111111111111) | ✅ (Invalidate) | ❌ |
| **Export/import** | ✅ (with slot picker) | 🟡 | 🟡 |
| **Max companion slots** | ✅ 18 | ✅ 18 | 🟡 6 |
| **Creature Builder (Web) link** | ✅ (button at top of panel) | ❌ | ❌ |

### Creature Part Descriptors (NMSE Unique)

NMSE includes a CreaturePartDatabase with 3056 descriptor nodes across 55 creature types, allowing users to customize creature appearance (body parts, colours, markings). This feature is not available in either NomNom or NMSSaveEditor.

## Settlement Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Settlement name** | ✅ | ✅ | 🟡 |
| **Seed value** | ✅ (validated hex) | ✅ | 🟡 |
| **Owner matching** | ✅ (LID/UID/USN) | ✅ | ✅ |
| **Population** | ✅ | ✅ | ❌ (broken) |
| **Stats (Happiness, Productivity, etc.)** | ✅ (8 stats including BugAttack) | ✅ (7 stats) | 🟡 |
| **Perks** | ✅ | ✅ | 🟡 |
| **Production state** | ✅ | ✅ | 🟡 (read only) |
| **Pending judgement type** | ✅ | ✅ | ❌ |
| **Last judgement time** | ✅ | ✅ | ❌ |
| **Delete settlement** | ✅ (in-place clear, removes from filter) | ✅ (Invalidate) | ❌ |
| **Import settlement** | ✅ (with slot picker, append/overwrite) | 🟡 | ❌ |
| **Export settlement** | ✅ | 🟡 | ❌ |
| **Max settlement slots** | ✅ 100 | ✅ 100 | 🟡 ? |

### Settlement Delete Approach

NMSE clears all settlement fields in-place (Name, SeedValue, Owner LID/UID/USN, Stats, Population, Perks, ProductionState, PendingJudgementType, LastJudgementTime). Critically clears Owner so FilterSettlements no longer matches the player. NomNom uses array entry removal (Invalidate).

## Base Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Storage container inventories (0-9)** | ✅ | ✅ | ✅ |
| **Nutrient Processor inventory** | ✅ | ✅ | ❌ |
| **Fish Platform inventory** | ✅ | ❌ | 🟡 |
| **Fish Bait Box inventory** | ✅ | ❌ | ❌ |
| **NPC seed editing** | ✅ | ✅ | ✅ |
| **NPC worker summoning** | ✅ | ✅ | ❌ |

## Discovery Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Discovery list display** | ✅ | ✅ | 🟡 |
| **System discoveries** | ✅ | ✅ | 🟡 |
| **Planet discoveries** | ✅ | ✅ | 🟡 |
| **Fauna discoveries** | ✅ | ✅ | ❌ |
| **Flora discoveries** | ✅ | ✅ | ❌ |
| **Mineral discoveries** | ✅ | ✅ | ❌ |
| **Race array** | ✅ (larger) | ✅ | ❌ |

## Milestone Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Milestone progress display** | ✅ | ✅ | 🟡 |
| **Milestone value editing** | ✅ | ✅ | 🟡 |
| **Milestone types** | ✅ (all categories) | ✅ | 🟡 |

## ByteBeat Song Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Song list display** | ✅ | ❌ | ❌ |
| **Song data editing** | ✅ | ❌ | ❌ |
| **Export/import songs** | ✅ | ❌ | ❌ |

## Exosuit Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **General inventory** | ✅ | ✅ | ✅ |
| **Technology inventory** | ✅ | ✅ | ✅ |
| **Cargo inventory** | ✅ | ✅ | ✅ |
| **Inventory resize** | ✅ | ✅ | ✅ |
| **Supercharge slots** | ✅ | ✅ | ✅ |
