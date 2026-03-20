# Editor Comparison: NMSE vs NomNom vs NMSSaveEditor

### Produced and correct at time of launch
_Probably won't be true for long considering it's open source 😜_

## Overview

This document set compares the three known No Man's Sky save editors:

| | **NMSE** | **NomNom** | **NMSSaveEditor** |
|---|---|---|---|
| **Platform** | .NET 10 WinForms | .NET WPF (MVVM) | Java Swing |
| **Language** | C# | C# | Java (obfuscated) |
| **License** | Open source | Mixed (editor: Closed, lib: Open) | Closed source |
| **Status** | Active development | Maintained (regular updates) | Unmaintained (out of date, sometimes updated) |
| **OS (Native)** | Windows | Windows | Windows/Mac/Linux (Java) |
| **Data Extractor** | Yes (NMSE.Extractor) | ? (Not Public) | ? (Not Public) |
| **Test Suite** | 1153 unit tests | Unknown | Unknown |

---

## Feature Summary

| Feature Area | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| Player Stats (Units, Nanites, QS, Health) | ✅ | ✅ | ✅ |
| Exosuit Inventory Editing | ✅ | ✅ | ✅ |
| Starship Editing (Seed, Type, Class, Stats) | ✅ | ✅ | ✅ |
| Starship Customisation Data | ✅ | ✅ | ❌ |
| Multitool Editing | ✅ | ✅ | ✅ |
| Freighter Editing | ✅ | ✅ | ✅ |
| Frigate Editing | ✅ | ✅ | 🟡 |
| Squadron Editing | ✅ | ✅ | 🟡 |
| Companion / Pet Editing | ✅ | ✅ | 🟡 |
| Creature Part Descriptors | ✅ | ❌ | ❌ |
| Settlement Editing | ✅ | ✅ | 🟡 |
| Base Editing (Storage, Chests) | ✅ | ✅ | 🟡 |
| Discovery Editing | ✅ | ✅ | 🟡 |
| Milestone Editing | ✅ | ✅ | 🟡 |
| ByteBeat Song Editing | ✅ | ❌ | ❌ |
| Recipe Browser | ✅ | ❌ | ❌ |
| Raw JSON Tree Editor | ✅ | ✅ | ✅ |
| Account Data Editing | ✅ | ✅ | 🟡 |
| Export/Import Individual Entities | ✅ | 🟡 | 🟡 |
| Coordinate / Glyph Display | ✅ | ✅ | ❌ |
| Difficulty Preset Editing | ✅ | ✅ | ❌ |
| Data Extractor Pipeline | ✅ | ❌ | ❌ |
| Custom Export Config | ✅ | ❌ | ❌ |

**Legend:** `✅` = Full support, `🟡` = Partial support, `❌` = Not supported

---

## Detailed Comparison Documents

- [Architecture Comparison](architecture.md) - Technology, project structure, testing
- [Save File I/O](save-io.md) - Read/write pipelines, formats, encryption
- [Inventory and Items](features-inventory.md) - Inventory grids, tech, supercharge
- [Vehicles and Equipment](features-vehicles.md) - Ships, multitools, exocraft, freighter, frigates
- [Player and World](features-player.md) - Stats, companions, settlements, discoveries
- [Extra Features](features-extras.md) - Data extractor, recipe browser, raw JSON, export/import