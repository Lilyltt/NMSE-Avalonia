# NMSE.Extractor

## Overview

NMSE.Extractor (`NMSE.Extractor/`) is a .NET console application that mines the No Man's
Sky game archives to produce the JSON databases, icons, and code that the main editor uses
at runtime. It runs a 12-stage pipeline that:

1. Locates the NMS installation via the Steam registry
2. Extracts selected files from `.pak` archives using `hgpaktool`
3. Converts binary `.mbin` files to XML-like `.mxml` using `MBINCompiler`
4. Parses `.mxml` into structured dictionaries
5. Categorizes items into typed JSON database files
6. Converts `.dds` textures to `.png` icons

The output lands in `Resources/db/` (JSON databases), `Resources/json/lang/` (per-language
localisation files), and `Resources/icons/` (PNG icons), ready for the main application to
load.

---

## Pipeline Stages

```
Stage  1: SteamLocator.FindPcBanksPath()
           -> Locate NMS GAMEDATA/PCBANKS directory

Stage  2: Validate storage
           -> Ensure enough disk space for extraction

Stage  3: ToolManager
           -> Download/update hgpaktool, MBINCompiler, ImageMagick, 7zip from GitHub

Stage  4: Cleanup
           -> Remove stale extracted files from previous runs

Stage  5: PakExtractor
           -> Extract filtered files from .pak archives (MBIN + DDS textures)

Stage  6: MBIN consolidation
           -> Gather all extracted .mbin files into a working directory

Stage  7: MbinConverter
           -> Convert .mbin -> .mxml using MBINCompiler

Stage  8: LocalisationBuilder
           -> Parse per-language MXML files, build lang/{bcp47}.json for all 16 languages
              (unescaped UTF-8 via JavaScriptEncoder.UnsafeRelaxedJsonEscaping)

Stage  9: Parsers
           -> Parse all .mxml files into typed dictionaries

Stage 10: Categorizer + JsonWriter
           -> Assign items to categories, write JSON database files

Stage 11: Code generation
           -> Generate TechPackDatabase.Generated.cs

Stage 12: ImageExtractor
           -> Convert DDS textures to PNG icons via ImageMagick
```

---

## Classes

### Program

| | |
|---|---|
| File | `NMSE.Extractor/Program.cs` |
| Purpose | Entry point that orchestrates the 12-stage extraction pipeline |

The `Main` method runs each stage sequentially, logging progress to the console (and to a
log file via `TeeTextWriter`). Each stage is wrapped in error handling -- failures in
non-critical stages log warnings and continue; failures in critical stages (missing required
MXML files) abort the run.

---

### ExtractorConfig

| | |
|---|---|
| File | `NMSE.Extractor/ExtractorConfig.cs` |
| Purpose | Central configuration -- tool URLs, file filters, required output paths |

A static class holding constants:

| Constant | Description |
|----------|-------------|
| `NmsGamePath` | Relative path: `steamapps\common\No Man's Sky\GAMEDATA\PCBANKS` |
| `HgPakToolZipUrl` / `HgPakToolLatestUrl` | GitHub release URLs for the PAK extraction tool |
| `MbinCompilerUrl` / `MbinCompilerLatestUrl` | GitHub release URLs for the MBIN-to-MXML compiler |
| `ImageMagickLatestUrl` / `ImageMagickDownloadPattern` | ImageMagick download URLs |
| `SevenZipLatestUrl` / `SevenZipDownloadPattern` | 7zip download URLs |
| `MappingJsonUrl` | URL for the MBIN field name mapping file |
| `MbinFilters` | 19 glob patterns selecting which MBIN files to extract (products, recipes, technology, rewards, language, etc.). Includes wildcard locale patterns (`nms_loc1_*.mbin`) to capture all language variants |
| `TextureFilters` | DDS texture file patterns |
| `AllExtractionFilters` | Combined MBIN + texture filters |
| `RequiredMxmlFiles` | MXML files that must exist after conversion (fatal if missing) |
| `OptionalMxmlFiles` | Non-English locale MXML files; language packs may not all be present |
| `SupportedLanguages` | Dictionary of 16 languages mapping language names to BCP 47 tags (English->en-GB, French->fr-FR, Italian->it-IT, German->de-DE, Spanish->es-ES, Russian->ru-RU, Polish->pl-PL, Dutch->nl-NL, Portuguese->pt-PT, LatinAmericanSpanish->es-419, BrazilianPortuguese->pt-BR, SimplifiedChinese->zh-CN, TraditionalChinese->zh-TW, Korean->ko-KR, Japanese->ja-JP, USEnglish->en-US). TencentChinese is excluded. |
| `LocaleFileStems` | 8 file stems for locale data: `nms_loc1`, `loc4`–`loc9`, `nms_update3` |
| `LangSubfolder` | Constant `"lang"` - the subfolder used for per-language JSON output |

| Method | Description |
|--------|-------------|
| `GetLocaleMxmlFiles(language)` | Returns the set of MXML file paths for a specific language by combining `LocaleFileStems` with the language suffix |

---

### PakExtractor

| | |
|---|---|
| File | `NMSE.Extractor/PakExtractor.cs` |
| Purpose | Extract files from NMS `.pak` archives using the external `hgpaktool` |

Iterates over `.pak` files in the PCBANKS directory, invoking `hgpaktool` with the filter
patterns from `ExtractorConfig.AllExtractionFilters`. Only extracts files matching the
filters to minimize disk usage and extraction time.

---

### MbinConverter

| | |
|---|---|
| File | `NMSE.Extractor/MbinConverter.cs` |
| Purpose | Batch-convert `.mbin` files to `.mxml` using the external `MBINCompiler` |

Runs `MBINCompiler` on all extracted `.mbin` files. The compiler decompiles NMS's binary
data format into an XML-like structure (`.mxml`) that can be parsed as text.

---

### MxmlParser

| | |
|---|---|
| File | `NMSE.Extractor/MxmlParser.cs` |
| Purpose | Parse `.mxml` files into dictionaries |

The MXML format uses nested `<Property>` elements with `name`, `value`, and `_index`
attributes. This parser reads the XML and produces `Dictionary<string, object>` trees that
the `Parsers` class consumes.

`LoadLocalisation` loads the English localisation lookup from `lang/en-GB.json`. If the new
per-language file is not found, it falls back to the legacy `Localisation-EN.json.tbc` file
for backwards compatibility.

---

### LocalisationBuilder

| | |
|---|---|
| File | `NMSE.Extractor/LocalisationBuilder.cs` |
| Purpose | Build per-language localisation JSON files from NMS language MXML files |

NMS ships each language as a set of MXML files where only the target language property
contains values (e.g., `nms_loc1_japanese.MXML` has data only in the `Japanese` property).

`BuildLocalisationJson()` iterates all 16 supported languages defined in
`ExtractorConfig.SupportedLanguages`. For each language it:

1. Reads the language's MXML file set via `ExtractorConfig.GetLocaleMxmlFiles(language)`
2. Calls `ParseLocalisation(mxmlPath, languageName)` to extract key-value pairs from the
   specified language property
3. Writes the merged dictionary to `Resources/json/lang/{bcp47}.json`

The resulting `en-GB.json` replaces the previous `Localisation-EN.json.tbc` file. All 16
language files live under the `lang/` subfolder, keyed by BCP 47 tag.

---

### Parsers

| | |
|---|---|
| File | `NMSE.Extractor/Parsers.cs` |
| Purpose | MXML-to-JSON parsers for all game data types |

A large static class containing 20+ specialized parser functions. Each parser reads a
specific MXML file type and produces a list of standardized dictionaries. Examples:

| Parser | Input MXML | Output |
|--------|-----------|--------|
| `ParseProductTable()` | Product definitions | Items with prices, crafting data, stats |
| `ParseRecipeTable()` | Recipe definitions | Recipes with ingredients and results |
| `ParseTechnologyTable()` | Technology definitions | Tech modules with stats and requirements |
| `ParseSubstanceTable()` | Substance definitions | Elements and resources |
| `ParseRewardTable()` | Reward definitions | Season/Twitch/platform rewards |
| `ParseLocalisationTable()` | Language files | Key-to-string translations |
| `ParseCreatureParts()` | Creature descriptors | Appearance customization data |
| `ParseWords()` | Alien speech table | Word entries with per-race group mappings and `Text_LocStr` |
| `ParseAllRecipes()` | Recipe table | Unified refining + cooking recipes |
| `ParseTitles()` | `PLAYERTITLEDATA.MXML` | Player titles with {0} name placeholder and localisation keys |
| `ParseFrigateTraits()` | `FRIGATETRAITTABLE.MXML` | Frigate traits with numeric Strength, Beneficial, Primary/Secondary, and localisation keys |
| `ParseSettlementPerks()` | `SETTLEMENTPERKSTABLE.MXML` | Settlement perks with stat changes and localisation keys |
| `ParseWikiGuide()` | `WIKI.MXML` | Wiki guide topics with IconKey, organized by category with localisation keys |

Helper methods include `BuildCanonicalProductDict()` for standardized field ordering and
`NullIfEmpty()` for empty-string-to-null conversion.

**Localisation string enrichment:** All parser outputs now include `_LocStr` keys alongside
the resolved English text fields. These keys hold the raw MXML localisation key values so
that consumers can look up translated text from the per-language JSON files:

| Key | Source |
|-----|--------|
| `Name_LocStr` | Raw MXML `Name` property value (e.g. `"UI_FUEL_1_NAME"`) |
| `NameLower_LocStr` | Raw MXML `NameLower` property value |
| `Subtitle_LocStr` | Raw MXML `Subtitle` property value |
| `Description_LocStr` | Raw MXML `Description` property value |
| `UnlockDescription_LocStr` | Title unlock description key |
| `AlreadyUnlockedDescription_LocStr` | Title already-unlocked description key |
| `RecipeName_LocStr` | Recipe name localisation key |
| `RecipeType_LocStr` | Recipe type localisation key |
| `Category_LocStr` | Category/heading localisation key (wiki guides, items) |
| `Text_LocStr` | Word text localisation key (first group key by race ordinal, e.g. `"TRA_ABOMINATION"`) |

---

### Categorizer

| | |
|---|---|
| File | `NMSE.Extractor/Categorizer.cs` |
| Purpose | Assign parsed items to output JSON category files |

Takes the flat list of parsed items and sorts them into 15+ category files based on item
group, name patterns, and regex rules.

**Output categories include:**

| File | Content |
|------|---------|
| `Food.json` | Carnivore Bait, Edible Product, Raw Ingredient |
| `Constructed Technology.json` | 50+ technology types |
| `Buildings.json` | 100+ building and furniture types |
| `Starship.json` | Spacecraft categories |
| `Upgrades.json` | Ship/weapon/suit upgrade modules |
| `Raw Materials.json` | Resources and elements |
| `Rewards.json` | Expedition, Twitch, and platform rewards |
| `Recipes.json` | Crafting and refining recipes |
| `Words.json` | Alien word translations with per-race group mappings |
| `Corvette.json` | Corvette building parts |
| `Curiosities.json` | Curiosities, Fossils, etc. |
| `Exocraft.json` | Exocraft parts |
| `Fish.json` | Fish |
| `Products.json` | Products |
| `Technology Module.json` | Unusual technology modules |
| `Technology.json` | Technologies |
| `Trade.json` | Trade goods |
| `Others.json` | Trails, Twitch Items, Charts, etc. |
| `none.json` | Items needing further categorisation (Ruins, Logs, etc.) |
| `Titles.json` | Player titles with localisation (318 entries) |
| `FrigateTraits.json` | Frigate traits with stat types (178 entries) |
| `SettlementPerks.json` | Settlement perks with stat changes (90 entries) |
| `WikiGuide.json` | Wiki guide topics by category (57 entries) |
| `TechPackDatabase.Generated.cs` | Procedural tech entries for future hashing |
| `Egg Modifiers.json.tbc` | Egg sequencing info (unimplemented) |
| `lang/en-GB.json` | English localisation strings (replaces legacy `Localisation-EN.json.tbc`) |
| `lang/fr-FR.json` | French localisation strings |
| `lang/ja-JP.json` | Japanese localisation strings |
| `lang/*.json` | … and 13 more per-language localisation files (see `ExtractorConfig.SupportedLanguages`) |


Classification uses:
- **Exact rules** -- `ExactRules` dictionary maps output filename to sets of item groups
- **Regex patterns** -- `TechModuleClassPattern`, `StarshipComponentGroupPattern`, etc.
- **Inclusion/exclusion sets** -- `StarshipExactGroups`, `StarshipExcludedGroups`

---

### JsonWriter

| | |
|---|---|
| File | `NMSE.Extractor/JsonWriter.cs` |
| Purpose | Serialize categorized data into formatted JSON database files |

Writes the categorized item lists, recipes, and rewards to JSON files in `Resources/db/`.
Uses consistent formatting for readable diffs.

---

### ImageExtractor

| | |
|---|---|
| File | `NMSE.Extractor/ImageExtractor.cs` |
| Purpose | Convert DDS textures to PNG icons using ImageMagick |

NMS stores item icons as `.dds` (DirectDraw Surface) textures. This class invokes
ImageMagick to convert them to `.png` files in `Resources/icons/`. The resulting PNGs are
loaded by `IconManager` at runtime.

---

### ProductLookup

| | |
|---|---|
| File | `NMSE.Extractor/ProductLookup.cs` |
| Purpose | Cross-reference products during parsing for dependency resolution |

Used by `Parsers` when an item references another item by ID. Maintains an in-memory lookup
of all parsed products so that parsers can resolve display names, icons, and categories for
referenced items.

---

### TeeTextWriter

| | |
|---|---|
| File | `NMSE.Extractor/TeeTextWriter.cs` |
| Purpose | Duplicate console output to a log file |

A `TextWriter` subclass that writes to both `Console.Out` and a file stream simultaneously.
Installed as `Console.SetOut(new TeeTextWriter(...))` at pipeline start so all output is
logged for debugging.

---

### SteamLocator

| | |
|---|---|
| File | `NMSE.Extractor/SteamLocator.cs` |
| Purpose | Locate the NMS game installation via the Windows registry |

| Method | Description |
|--------|-------------|
| `FindPcBanksPath()` | Returns the full path to `GAMEDATA\PCBANKS` |
| `GetSteamInstallPath()` | Reads Steam install directory from registry (checks 64-bit first) |
| `ReadRegistryValue(subKey, valueName)` | Helper for Windows registry queries |

Used in Stage 1 of the pipeline to find the game files without user input.

---

### ToolManager

| | |
|---|---|
| File | `NMSE.Extractor/ToolManager.cs` |
| Purpose | Download and version-manage external tools from GitHub releases |

Manages `hgpaktool`, `MBINCompiler`, `ImageMagick`, and `7zip`. For each tool:

1. Check local version file against GitHub's latest release tag
2. If outdated or missing, download the release asset
3. Extract and write a version file for caching

| Method | Description |
|--------|-------------|
| `GetLatestReleaseTagAsync(latestUrl)` | Resolve GitHub `/releases/latest/` redirect to actual tag |
| `ReadVersionFile(path)` | Read cached version string |
| `WriteVersionFile(path, tag)` | Write version string |

Uses two `HttpClient` instances: one with auto-redirect (for downloads) and one without (for
tag resolution via `Location` header).

---

## Data Flow Diagram

```
NMS Game Files (.pak archives)
    |
    v
PakExtractor (hgpaktool)
    |
    +-- .mbin files -----> MbinConverter (MBINCompiler) ----> .mxml files
    |                                                            |
    +-- .dds textures ---> ImageExtractor (ImageMagick) -> .png icons
                                                            |
                                                            v
                                                    Resources/icons/
                           .mxml files
                             |
                             v
                    LocalisationBuilder --> lang/{bcp47}.json (×16 languages)
                             |                     |
                             v                     v
                          Parsers            Resources/json/lang/
                  (enriched with _LocStr)
                             |
                             v
                   Categorizer + JsonWriter
                             |
                             v
                      Resources/db/*.json
                             |
                             v
              TechPackDatabase.Generated.cs (code gen)
```

---

## Testing

`NMSE.Extractor.Tests/` contains unit tests for the extractor's parsing and categorization
logic. Tests verify that MXML parsing produces expected dictionary structures, that
categorization rules assign items to correct output files, and that edge cases (empty fields,
missing attributes) are handled gracefully.
