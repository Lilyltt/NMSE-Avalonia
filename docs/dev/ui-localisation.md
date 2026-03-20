# UI Localisation

## Overview

NMSE supports UI string localisation for menus, tabs, dialog messages, toolbar labels,
status bar text, panel labels, section headers, grid column headers, context menus,
and all other user-visible controls. This is separate from the game-data
localisation (item names, descriptions, etc.) which uses NMS internal localisation keys.

## Architecture

### UiStrings Static Class

The `Data/UiStrings.cs` class provides:

| Method | Description |
|--------|-------------|
| `SetDirectory(path)` | Sets the directory containing UI string JSON files |
| `Load(bcp47Tag)` | Loads a language (always loads `en-GB.json` as fallback) |
| `Get(key)` | Returns the localised string, English fallback, or raw key |
| `GetOrNull(key)` | Returns the localised string or `null` if missing |
| `Format(key, args)` | Returns the localised string with `string.Format()` arguments |
| `TotalKeyCount` | Number of keys defined in the English fallback |
| `TranslatedCount` | Number of keys in the active (non-English) language |
| `IsActive` | Whether a non-English language is currently loaded |
| `Reset()` | Clears all state (used in unit tests) |

### File Location

UI string JSON files are stored in `Resources/ui/lang/{bcp47}.json`:

```
Resources/
  ui/
    lang/
      en-GB.json    ← Source of truth (English)
      fr-FR.json    ← French
      de-DE.json    ← German
      ja-JP.json    ← Japanese
      ... (16 files total)
```

### Key Format

Keys use dot-delimited hierarchical naming:

```json
{
  "menu.file": "&File",
  "menu.file.save": "&Save",
  "tab.player": "&Player",
  "player.health": "Health:",
  "status.loaded_save": "Loaded: {0} ({1}ms)",
  "dialog.save_success": "Save file written successfully!",
  "settlement.perk": "Perk {0}:",
  "bytebeat.data_channel": "Data [{0}]:"
}
```

#### Key Prefixes

| Prefix | Scope |
|--------|-------|
| `menu.*` | Main menu items (File, Edit, Tools, Language, Help) |
| `tab.*` | Top-level tab page titles |
| `toolbar.*` | Toolbar labels |
| `status.*` | Status bar messages |
| `dialog.*` | Dialog/message box messages, file dialog titles and filters |
| `common.*` | Shared strings (Cargo, Technology, filter, Unknown, Max Supported, etc.) |
| `player.*` | Player panel labels, section headers, coordinates |
| `starship.*` | Starship panel labels and buttons |
| `multitool.*` | Multi-tool panel labels and buttons |
| `freighter.*` | Freighter panel labels |
| `settlement.*` | Settlement panel labels and stats |
| `companion.*` | Companion panel labels, traits, seeds |
| `exocraft.*` | Exocraft panel labels |
| `frigate.*` | Frigate panel labels, stats, traits |
| `squadron.*` | Squadron panel labels |
| `discovery.*` | Discovery panel labels, tabs, columns |
| `milestone.*` | Milestone panel labels and sections |
| `bytebeat.*` | ByteBeat panel labels and sections |
| `account.*` | Account/rewards panel |
| `base.*` | Base panel tabs |
| `inventory.*` | Inventory grid labels, context menu |
| `item_picker.*` | Item picker dialog buttons, columns, placeholders |
| `recipe.*` | Recipe panel columns |
| `raw_json.*` | JSON editor labels, context menu |
| `export_config.*` | Export config tabs and buttons |
| `fleet.*` | Fleet panel tabs |
| `exosuit.*` | Exosuit tabs |
| `app.*` | Application-level strings |

### Integration Points

1. **`MainForm.LoadDatabase()`** – Sets the UI lang directory via `UiStrings.SetDirectory()`
2. **`MainForm.ApplyStartupLanguage()`** – Calls `UiStrings.Load()` and `ApplyUiLocalisation()`
3. **`MainForm.OnLanguageSelected()`** – Reloads UI strings on language switch
4. **`MainForm.ApplyUiLocalisation()`** – Pushes translated strings to:
   - Main menu items (File, Edit, Tools, Language, Help)
   - Toolbar labels (Directory, Save Slot, File, Browse, Load, Save)
   - Tab page titles (Player, Exosuit, Multi-tools, etc.)
5. **Panel `ApplyUiLocalisation()` methods** – Each of the 20 panels has its own
   `ApplyUiLocalisation()` method that updates:
   - Form labels (e.g. "Health:", "Shield:", "Name:", stat labels)
   - Section headers (e.g. "Player Statistics", "Song Details")
   - Buttons (Export, Import, Generate, etc.)
   - Tab sub-pages (General, Cargo, Technology, etc.)
   - Grid column headers
   - Context menu items
   - Filter/search placeholders
   - Dynamic count labels (via `UiStrings.Format()`)
6. **Status bar** – UI string count is included in the `_itemCountLabel` total
7. **Language status** – UI string count shown in language-switch status message

### Panel Label Pattern

Labels created by `AddRow()` or `AddSectionHeader()` in `*.Designer.cs` files return
the `Label` reference, which is stored as a private field. The corresponding
`ApplyUiLocalisation()` method updates these labels on language switch:

```csharp
// In Designer.cs – store the label reference:
_healthLabel = AddRow(leftLayout, "Health:", _healthField, leftRow++);

// In Panel.cs – update on language change:
public void ApplyUiLocalisation()
{
    _healthLabel.Text = UiStrings.Get("player.health");
}
```

### Accelerator Keys (Mnemonics)

For `en-GB` and `en-US`, menu items and tab page labels include `&` mnemonic markers
that underline a letter for keyboard navigation:

```json
{
  "menu.file": "&File",        // Underlines 'F'
  "tab.player": "&Player",    // Underlines 'P'
  "tab.starships": "&Starships" // Underlines 'S'
}
```

Non-English languages typically omit the `&` prefix since accelerator keys are
language-specific and may not apply to translated strings.

### Safety

- **No logic impact:** UI strings are display-only. All filtering, comparison, and save
  I/O logic uses English internal values from the data layer.
- **Fallback chain:** Active language -> English fallback -> raw key string
- **Format safety:** `UiStrings.Format()` catches `FormatException` from mismatched
  placeholders and returns the unformatted template.

## Supported Languages

| BCP 47 | Language |
|--------|----------|
| en-GB | English (Great Britain) – Source |
| en-US | English (United States) |
| fr-FR | French |
| de-DE | German |
| it-IT | Italian |
| es-ES | Spanish (Spain) |
| es-419 | Latin American Spanish |
| pt-PT | Portuguese (Portugal) |
| pt-BR | Brazilian Portuguese |
| nl-NL | Dutch |
| pl-PL | Polish |
| ru-RU | Russian |
| ja-JP | Japanese |
| ko-KR | Korean |
| zh-CN | Simplified Chinese |
| zh-TW | Traditional Chinese |

## Adding New Strings

1. Add the English string to `Resources/ui/lang/en-GB.json`
2. Add corresponding translations to all other 15 language files
3. Reference the key in code via `UiStrings.Get("your.key")` or `UiStrings.Format("your.key", args)`
4. For panel labels: store the `Label` reference and add an update call in `ApplyUiLocalisation()`
5. For `en-GB`/`en-US`: include `&` mnemonic prefixes for menu/tab items (e.g. `"&File"`)

### Current Key Count

All 16 language files contain **1158 keys** each (0 missing).

## Audit Record

A class-by-class audit of every C# file is maintained in
[`localisation-audit.md`](localisation-audit.md). This covers all files in `UI/`,
`Core/`, and `Data/` directories and documents which strings are localised, which
were addressed, and which are intentionally excluded (internal identifiers, proper
nouns, single-character grades).

## Testing

Unit tests are in `NMSE.Tests/UiStringsTests.cs` (17 tests) covering:
- Loading and fallback behaviour
- Language switching
- Format placeholder substitution
- Error handling for missing files/keys
- TranslatedCount and TotalKeyCount properties

Logic tests in `NMSE.Tests/LogicTests.cs` load `en-GB.json` via `UiStrings` so that
assertions on localised fallback strings (e.g. `"Unknown"`, `"Multitool 1"`) continue
to pass.
