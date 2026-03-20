# Inventory and Items Comparison

## Inventory Grid Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Visual grid display** | ✅ (InventoryGridPanel) | ✅ (WPF grid) | ✅ (Swing grid) |
| **Add items** | ✅ (item picker dialog) | ✅ (item picker) | ✅ |
| **Remove items** | ✅ (remove + reindex) | ✅ (JArray remove) | ✅ |
| **Edit stack size** | ✅ | ✅ | ✅ |
| **Resize inventory (W x H)** | ✅ (NUD controls) | ✅ (Height/Width) | 🟡 |
| **Supercharge slots** | ✅ | ✅ | 🟡 |
| **Item type filtering** | ✅ (TechnologyCategory) | ✅ (TechnologyCategory) | 🟡 |
| **Procedural item seed** | ✅ (appends #seed suffix) | ✅ (itemId#seed) | ❌ |

## Inventory Types Supported

| Inventory | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Exosuit General** | ✅ | ✅ | ✅ |
| **Exosuit Technology** | ✅ | ✅ | ✅ |
| **Exosuit Cargo** | ✅ | ✅ | ✅ |
| **Starship General** | ✅ | ✅ | ✅ |
| **Starship Technology** | ✅ | ✅ | ✅ |
| **Starship Cargo** | ✅ | ✅ | ✅ |
| **Multitool** | ✅ | ✅ | ✅ |
| **Multitool Technology** | ✅ | ✅ | ❌ |
| **Freighter General** | ✅ | ✅ | ✅ |
| **Freighter Technology** | ✅ | ✅ | 🟡 |
| **Freighter Cargo** | ✅ | ✅ | 🟡 |
| **Vehicle** | ✅ | ✅ | 🟡 |
| **Storage Containers (0-9)** | ✅ | ✅ | 🟡 |
| **Nutrient Processor** | ✅ | ✅ | ❌ |
| **Fish Platform** | ✅ | ❌ | ❌ |
| **Fish Bait Box** | ✅ | ❌ | ❌ |

## Supercharge Slots

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Toggle supercharge on slot** | ✅ | ✅ | ✅ |
| **Supercharge structure** | ✅ | ✅ | ❌ |
| **Type check correctness** | ✅ (more strict - checks type field) | 🟡 (less strict) | ❌ |
| **Exocraft restriction** | ✅ (disabled for exocraft) | ✅ | ❌ |

## Item Databases

| Database | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Game items (substances, products, tech)** | ✅ (GameItemDatabase from JSON) | ✅ (Record.cs from EXML) | 🟡 (built-in) |
| **TechnologyCategory field** | ✅ (added via extractor) | ✅ (built-in) | 🟡 |
| **IsUpgrade / IsCore / IsProcedural** | ✅ | ✅ | 🟡 |
| **DeploysInto** | ✅ | ✅ | 🟡 |
| **Tech pack identification (hashes)** | ✅ (TechPackDatabase, 736 entries) | ❌ | 🟡 |
| **Stack limit database** | ✅ (InventoryStackDatabase) | ✅ (via model) | 🟡 |
| **Recipe database** | ✅ (RecipeDatabase, 1680 recipes) | ✅ | ❌ |
| **Icons** | ✅ (IconManager) | 🟡 (embedded) | 🟡 |

## Slot Operations

### Adding Items

| Step | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Item picker** | ItemPickerDialog with search and category filtering | WPF picker with categories | Item picker with filtering |
| **Slot creation** | Creates full JSON slot with Type, Id, Amount, MaxAmount, Index, DamageFactor, FullyInstalled | Similar via InventorySlot model | Similar |
| **Procedural suffix** | Appends `#seed` for IsProcedural items | Appends `#seed` via ItemId setter | Appends `#seed` |

### Removing Items

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Removal method** | ✅ Remove from Slots array + reindex remaining cells | ✅ JArray remove | ✅ |
| **Index recomputation** | ✅ (fixes X/Y coordinates) | ✅ | ✅ |

### Resizing

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Width range** | ✅ 1-15 (panel-specific) | ✅ Varies by entity | ✅ |
| **Height range** | ✅ 1-13 (panel-specific) | ✅ Varies by entity | ✅ |
| **Overflow handling** | ✅ Removes out-of-bounds slots | ✅ Removes out-of-bounds slots | 🟡 |

## Base Stat Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Stat editing via NUDs** | ✅ | ✅ | ✅ |
| **Min/Max limits** | ✅ (BaseStatLimits.GetRange()) | ✅ (typed stat limits) | 🟡 |
| **Clamping on save** | ✅ (BaseStatLimits.ClampStatValue()) | ✅ (via model setters) | ❌ |
| **Multi-inventory stat writes** | ✅ (all 3 inventory variants) | ✅ (iterates Inventories) | 🟡 |

## Unique NMSE Features

- **TechPackDatabase**: 736 hash-to-ID mappings for post-Worlds tech identification (not available in NomNom, partially in NMSSaveEditor)
- **Fish Platform/Bait Box inventories**: Full editing for fishing features (partially available in NMSSaveEditor)
- **Custom export config**: Configurable file extensions and naming templates for import/export
