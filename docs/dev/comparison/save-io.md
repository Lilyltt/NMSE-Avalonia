# Save File I/O Comparison

## Platform Support

| Platform | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **PC (Steam)** | ✅ | ✅ | ✅ |
| **PC (GOG)** | ✅ | ✅ | ✅ |
| **PC (Microsoft Store)** | ✅ | ✅ | 🟡 |
| **PlayStation** | ✅ | ✅ | ✅ |
| **Xbox** | ✅ | ✅ | 🟡 |
| **Nintendo Switch** | ✅ | ✅ | 🟡 |

## Read Pipeline

### NMSE
1. `SaveSlotManager` reads `containers.index` to enumerate save slots
2. `ContainersIndexManager` parses the binary index format
3. `MemoryDatManager` reads `memory.dat` files
4. `MetaCrypto` decrypts the TEA-encrypted metadata header
5. `Lz4DecompressorStream` decompresses the LZ4 payload
6. `JsonParser` + `JsonReader` parse JSON into `JsonObject`/`JsonArray`
7. `JsonNameMapper` translates obfuscated keys to human-readable names
8. `SaveFileManager.RegisterContextTransforms()` maps ActiveContext/BaseContext to PlayerStateData

### NomNom (via libNOM.io)
1. Platform-specific container reading (Steam, GOG, MS Store, console)
2. LZ4 decompression via native interop
3. Newtonsoft.Json parsing into JObject
4. Built-in key deobfuscation
5. Context resolution built into model layer

### NMSSaveEditor
1. Custom binary reader for containers.index
2. Custom LZ4 decompression (Java)
3. Custom JSON parser
4. Built-in key mapping

## Write Pipeline

### NMSE
1. `SaveFileManager.SaveToFile()` orchestrates the write
2. `JsonNameMapper` re-obfuscates keys if mapper is active
3. `MetaFileWriter` writes the binary format:
   - TEA-encrypted metadata header via `MetaCrypto`
   - LZ4 compression via `Lz4Compressor` / `Lz4ChunkedCompressorStream`
4. `ContainersIndexManager` updates containers.index

### NomNom
1. libNOM.io handles write pipeline
2. Platform-specific container writing
3. LZ4 compression via native interop
4. Automatic key obfuscation

### NMSSaveEditor
1. Custom binary writer
2. Custom LZ4 compression
3. Custom JSON serialization

## Key Mapping (Obfuscation)

NMS uses obfuscated JSON keys in save files (e.g., `F2P` instead of `PlayerStateData`).

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Mapping source** | `JsonNameMapper` with dictionary from Resources | libNOM.io built-in | Built-in dictionary |
| **Direction** | Bidirectional (obfuscated <-> readable) | Bidirectional | Bidirectional |
| **Auto-detect** | Yes (checks key patterns) | Yes | Yes |
| **Export format** | Human-readable JSON | Human-readable JSON | Human-readable JSON |

## Import/Export

| Capability | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Export full save as JSON** | ✅ | ✅ | ✅ |
| **Import full save from JSON** | ✅ | ✅ | ✅ |
| **Export individual entity** | ✅ (ships, tools, companions, settlements, frigates, songs) | 🟡 (some entities) | 🟡 (limited) |
| **Import individual entity** | ✅ (with slot picker) | 🟡 | 🟡 |
| **NomNom format compatibility** | ✅ (reads .nmnom exports) | ✅ | ❌ |
| **NMSSaveEditor wrapping** | ✅ (detects outer wrapper) | ❌ | ✅ |
| **Custom file extensions** | ✅ (ExportConfig panel) | ❌ | ❌ |
| **Naming templates** | ✅ (configurable) | ❌ | ❌ |
