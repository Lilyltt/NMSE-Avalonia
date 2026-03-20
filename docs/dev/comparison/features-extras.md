# Extra Features Comparison

## Data Extractor (NMSE Unique)

NMSE includes a standalone data extraction tool (`NMSE.Extractor`) that processes No Man's Sky game files to produce the JSON databases used by the editor. Neither NomNom nor NMSSaveEditor include an equivalent **public** tool.

| Feature | NMSE.Extractor | NomNom | NMSSaveEditor |
|---|---|---|---|
| **PAK file extraction** | ✅ (PakExtractor) | ❌ | ❌ |
| **MBIN to MXML conversion** | ✅ (MbinConverter via MBINCompiler) | ❌ | ❌ |
| **MXML parsing** | ✅ (MxmlParser) | ❌ | ❌ |
| **Item categorization** | ✅ (Categorizer) | ❌ | ❌ |
| **JSON output generation** | ✅ (JsonWriter) | ❌ | ❌ |
| **Localization building** | ✅ (LocalisationBuilder) | ❌ | ❌ |
| **Image extraction** | ✅ (ImageExtractor) | ❌ | ❌ |
| **TechPack class generation** | ✅ (GenerateTechPackPartialClass) | ❌ | ❌ |
| **Recipe extraction** | ✅ (ParseAllRecipes) | ❌ | ❌ |
| **Title extraction** | ✅ (ParseTitles) | ❌ | ❌ |
| **TechnologyCategory enrichment** | ✅ (EnrichTechnologyCategory) | ❌ | ❌ |
| **Steam game path auto-detection** | ✅ (SteamLocator) | ❌ | ❌ |
| **Tool version management** | ✅ (ToolManager for MBINCompiler) | ❌ | ❌ |

### Extraction Pipeline

1. `PakExtractor` extracts PAK files from the NMS installation
2. `MbinConverter` converts MBIN files to MXML (human-readable XML)
3. `MxmlParser` parses MXML into structured data
4. `Categorizer` classifies items into categories (substances, products, technologies, upgrades)
5. `LocalisationBuilder` resolves localization strings for item names and descriptions
6. `JsonWriter` produces the final JSON databases consumed by the editor
7. `ImageExtractor` extracts and processes item icons

### Output (DB) Files

| File | Content | Used By |
|---|---|---|
| `Buildings.json` | Game items (building parts) | GameItemDatabase |
| `Constructed Technology.json` | Game items (constructed tech) | GameItemDatabase |
| `Corvette.json` | Game items (corvette parts) | GameItemDatabase |
| `Curiosities.json` | Game items (curiosities) | GameItemDatabase |
| `Exocraft.json` | Game items (exocraft parts) | GameItemDatabase |
| `Fish.json` | Game items (fish) | GameItemDatabase |
| `Food.json` | Game items (food) | GameItemDatabase |
| `Products.json` | Game items (products) | GameItemDatabase |
| `Raw Materials.json` | Game items (raw materials) | GameItemDatabase |
| `Starships.json` | Game items (starship parts) | GameItemDatabase |
| `Technology Module.json` | Game items (tech modules) | GameItemDatabase |
| `Technology.json` | Game items (technology) | GameItemDatabase |
| `Upgrades.json` | Game items (upgrades) | GameItemDatabase |
| `Trade.json` | Game items (trade goods) | GameItemDatabase |
| `Others.json` | Game items (other) | GameItemDatabase |
| `none.json` | Game items (needing categorisation) | GameItemDatabase |
| `Rewards.json` | Rewards | RewardDatabase |
| `Recipes.json` | 1680 crafting/refining/cooking recipes | RecipeDatabase |
| `Titles.json` | Player titles | TitleDatabase |
| `WikiGuide.json` | In game wiki/guide topics | WikiGuideDatabase |
| `Words.json` | Alien words | WordDatabase |
| `FrigateTraits.json` | Game Frigate Traits | FrigateTraitDatabase |
| `SettlementPerks.json` | Game Settlement Perks | SettlementPerkDatabase |
| `TechPackDatabase.Generated.cs` | Tech catalog entries with icons and categories | TechPackDatabase (partial class) |
| Various icon PNGs | Item and technology icons | IconManager |
| `Egg Modifiers.json.tbc` | Egg sequencing info | Currently Unimplemented |

### Output (Localisation) Files

16 JSON localisation files from the games localisation strings covering: English (UK), English (US), French, Italian, German, Spanish, Russian, Polish, Dutch, Portuguese, Latin American Spanish, Brazilian Portuguese, Simplified Chinese, Traditional Chinese, Japanese and Korean.

They are named per the the `IETF BCP 47` language tag specification, such as `en-GB.json`.

## Recipe Browser (NMSE Unique)

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Recipe browsing UI** | ✅ (RecipePanel) | ❌ | ❌ |
| **Recipe database** | ✅ (1680 recipes from extractor) | ✅ (but not exposed in UI) | ❌ |
| **Recipe categories** | ✅ (crafting, refining, cooking) | ❌ | ❌ |
| **Search/filter** | ✅ | ❌ | ❌ |

## Raw JSON Editor

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Tree view of save data** | ✅ (RawJsonPanel) | ✅ (JSON tree) | ✅ (JSON tree) |
| **Edit individual values** | ✅ | ✅ | ✅ |
| **Search within JSON** | ✅ | ✅ | ✅ |
| **Add/remove nodes** | ✅ | ✅ | ✅ |
| **Context-aware paths** | ✅ (respects context transforms) | ✅ (via libNOM.io) | 🟡 |

## Export / Import System

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Export full save as JSON** | ✅ | ✅ | ✅ |
| **Import full save from JSON** | ✅ (with NameMapper + context transforms) | ✅ | ✅ |
| **Export/Import individual starship** | ✅ | 🟡 | 🟡 |
| **Export/Import individual multitool** | ✅ | 🟡 | 🟡 |
| **Export/Import individual companion** | ✅ | 🟡 | 🟡 |
| **Export/Import individual settlement** | ✅ | 🟡 | ❌ |
| **Export/Import individual frigate** | ✅ | 🟡 | ❌ |
| **Export/Import ByteBeat song** | ✅ | ❌ | ❌ |
| **Export/Import squadron pilot** | ✅ | 🟡 | ❌ |
| **Import with slot picker** | ✅ (ItemPickerDialog for target slot) | ❌ | ❌ |
| **NomNom format compatibility** | ✅ (reads .nmnom files) | ✅ (native) | ❌ |
| **NMSSaveEditor wrapper detection** | ✅ (strips outer wrapper) | ✅ | ✅ (native) |
| **Custom file extensions** | ✅ (ExportConfigPanel) | ❌ | ❌ |
| **Custom naming templates** | ✅ (FileNameHelper) | ❌ | ❌ |

### Import Robustness

NMSE's import pipeline includes several safety features:
- **NameMapper restoration**: Ensures obfuscated keys are used on save even when importing human-readable JSON
- **Context transform registration**: Maps ActiveContext/BaseContext to PlayerStateData for modern save formats
- **Seed validation**: All imported seed values are validated as hex strings
- **Slot conflict resolution**: FindImportTargetIndex returns append, overwrite, or user-pick based on existing data

## Custom Export Configuration (NMSE Unique)

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **ExportConfig panel** | ✅ | ❌ | ❌ |
| **Custom file extensions per entity** | ✅ | ❌ | ❌ |
| **Naming templates** | ✅ (configurable patterns) | ❌ | ❌ |
| **Persistent configuration** | ✅ (AppConfig) | ❌ | ❌ |

## Coordinate Tools

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Glyph display** | ✅ | ✅ | ❌ |
| **Coordinate display** | ✅ | ✅ | ❌ |
| **Editable coordinates** | ✅ (6 NUD fields + Apply) | ✅ | ❌ |
| **Galaxy picker** | ✅ (GalaxyDatabase with 256 galaxies) | ✅ | ❌ |

## Wiki Integration (NMSE Unique)

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **WikiGuideDatabase** | ✅ | ✅ | ❌ |

## MXML Reward Editor (NMSE Unique)

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **MxmlRewardEditor** | ✅ (edit reward tables) | ❌ | ❌ |

## Testing

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Test framework** | xUnit | None visible | Unknown |
| **Test count** | 1153 | ~0 | Unknown |
| **Logic class coverage** | Comprehensive | N/A | N/A |
| **Save round-trip tests** | ✅ (load, modify, save, compare) | ❌ | ❌ |
| **Data layer tests** | ✅ (database integrity) | ❌ | ❌ |
| **Extractor tests** | ✅ (NMSE.Extractor.Tests) | N/A | N/A |
