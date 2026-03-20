# Architecture Comparison

## Technology Stacks

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Runtime** | .NET 10 | .NET Framework / .NET 5+ | JRE 8+ |
| **UI Framework** | WinForms | WPF (MVVM) | Java Swing |
| **JSON Library** | Custom (JsonObject/JsonArray) | Newtonsoft.Json (JObject/JArray) | Custom (obfuscated) |
| **Compression** | Custom LZ4 (managed C#) | libNOM.io (native interop) | Custom LZ4 (Java) |
| **Architecture** | Logic/Panel separation | MVVM (ViewModel/View) | Monolithic panels |
| **Testing** | 1153 xUnit tests | Minimal | Unknown |
| **Build System** | MSBuild + auto-increment patch | MSBuild | Gradle/Maven (obfuscated) |

## Project Structure

### NMSE (5 projects)

| Project | Purpose |
|---|---|
| NMSE | Main WinForms editor application |
| NMSE.Tests | 904 unit tests covering all logic classes |
| NMSE.Extractor | Standalone tool for extracting game data from PAK files |
| NMSE.Extractor.Tests | Tests for the extractor pipeline |
| NMSE.Site | Static GitHub Pages site for downloads |

### NomNom (2 main projects)

| Project | Purpose |
|---|---|
| NomNom | WPF application with MVVM architecture |
| libNOM.io | Separate library for save file I/O (NuGet package) |

### NMSSaveEditor (1 project)

Single monolithic Java project with obfuscated class names (697 classes post-decompilation).

## Design Patterns

### NMSE: Logic/Panel Separation
- **Logic classes** (`Core/*Logic.cs`): Pure data manipulation, no UI dependencies, fully testable
- **Panel classes** (`UI/Panels/*Panel.cs`): UI binding, event handlers, display formatting
- **Designer files** (`*Panel.Designer.cs`): VS Designer-compatible control initialization
- **Data classes** (`Data/*.cs`): Static databases, name mappings, game item definitions

### NomNom: MVVM
- **ViewModels** (`NomNom.ViewModel/`): Data binding with INotifyPropertyChanged
- **Views** (`NomNom.Views/`): XAML-based UI
- **Models** (`NomNom.Models/`): Entity wrappers around JObject nodes

### NMSSaveEditor: Monolithic
- Obfuscated class names make architecture analysis difficult
- Tab-based panel system similar to NMSE but without clear layer separation

## JSON Models

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Model** | Custom JsonObject/JsonArray | Newtonsoft JObject/JArray | Custom obfuscated |
| **Name mapping** | JsonNameMapper (bidirectional) | libNOM.io (built-in) | Built-in obfuscation |
| **Context transforms** | RegisterContextTransforms() | libNOM.io handles | Unknown |
| **Mutation style** | Direct Set()/Get() | JToken property access | Direct methods |
| **Parser** | Custom streaming JsonReader | Newtonsoft JsonConvert | Custom parser |

## Testing

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Framework** | xUnit | None visible | Unknown |
| **Test count** | 1153 | Unknown | Unknown |
| **Logic coverage** | All Logic classes | N/A | N/A |
| **Round-trip tests** | Save load/save/compare | No | No |
| **Data layer tests** | Database integrity checks | No | No |
