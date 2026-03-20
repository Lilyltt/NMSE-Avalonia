# Vehicles and Equipment Comparison

## Starship Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Ship name** | ✅ | ✅ | ✅* |
| **Ship seed** | ✅ (validated hex) | ✅ (validated hex) | ✅* |
| **Ship type / Resource.Filename** | ✅ | ✅ | ✅* |
| **Ship class (C/B/A/S)** | ✅ (all 3 inventories) | ✅ (all inventories) | ✅* |
| **Legacy colours toggle** | ✅ | ✅ | ✅* |
| **Base stats (Damage, Shield, Hyperdrive, Maneuver)** | ✅ (all 3 inventories) | ✅ (all inventories) | 🟡 (single inventory)* |
| **Primary ship index** | ✅ | ✅ | ✅* |
| **Active ship selection** | ✅ (Seed[0] boolean) | ✅ | ✅* |
| **Delete / invalidate ship** | ✅ | ✅ | ✅* |
| **Inventory editing** | ✅ (General, Tech, Cargo) | ✅ | ✅* |
| **Max ship slots** | ✅ 12 | ✅ 12 | 🟡 9 |
| **Customisation data** | ✅ | ✅ | ❌ |
| **Export/import individual ship** | ✅ (with slot picker) | 🟡 | 🟡* |

*NMSSaveEditor ship handling implementation is out of date due to customisation data and mangles ship data (sometimes silently, sometimes not).

### Starship Types Supported

| Type | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Normal (Fighter, Explorer, Hauler, Shuttle)** | ✅ | ✅ | ✅ |
| **Living Ship** | ✅ | ✅ | 🟡 |
| **Robot/Solar Ship** | ✅ | ✅ | 🟡 |
| **Corvette Ship** | ✅ (Includes part lookup) | 🟡 | ❌ |
| **Alien Ship** | ✅ | ✅ | 🟡 |
| **Unknown/Special Ship** | ✅ | ✅ | 🟡 |

## Multitool Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Tool name** | ✅ | ✅ | ✅ |
| **Tool seed** | ✅ (validated, syncs to CurrentWeapon) | ✅ (syncs when primary) | ✅ |
| **Tool type / Resource.Filename** | ✅ (nested path) | ✅ (nested path) | 🟡 |
| **Tool class** | ✅ (Store + Store_TechOnly) | ✅ | 🟡 |
| **Stats (Damage, Mining, Scan)** | ✅ (syncs WeaponInventory for primary) | ✅ (syncs WeaponInventory) | 🟡 |
| **Primary tool index** | ✅ | ✅ | ✅ |
| **Active tool selection** | ✅ | ✅ | ✅ |
| **Inventory editing** | ✅ (General, Technology) | ✅ | ✅ |
| **Max tool slots** | ✅ 6 | ✅ 6 | 🟡 3 |
| **Export/import individual tool** | ✅ | 🟡 | 🟡 |

### Multitool Types Supported

| Type | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Pistol** | ✅ | ✅ | ✅ |
| **Rifle** | ✅ | ✅ | ❌ |
| **Royal** | ✅ | ✅ | ✅ |
| **Alien** | ✅ | ✅ | ❌ |
| **Pristine** | ✅ | ✅ | ❌ |
| **Sentinel** | ✅ | ✅ | ✅ |
| **Sentinel B** | ✅ | ✅ | ✅ |
| **Switch** | ✅ | ✅ | ✅ |
| **Staff** | ✅ | ✅ | ✅ |
| **Staff NPC** | ✅ | ✅ | ✅ |
| **Staff Ruin** | ✅ | ✅ | ❌ |
| **Staff Bone** | ✅ | ✅ | ❌ |
| **Atlas** | ✅ | ✅ | ✅ |
| **Atlas Sceptre** | ✅ | ✅ | ✅ |

## Freighter Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Freighter name** | ✅ (syncs TeleportEndpoints) | ✅ (syncs TeleportEndpoints) | ✅ |
| **Freighter seed** | ✅ (validated hex) | ✅ | ✅ |
| **Freighter type / Filename** | ✅ | ✅ | ✅ |
| **Freighter class** | ✅ (all 3 inventories) | ✅ | 🟡 |
| **Base stats (Hyperdrive, FleetCoordination)** | ✅ (all 3 inventories) | ✅ (all inventories) | 🟡 |
| **Crew race** | ✅ (CurrentFreighterNPC.Filename) | ✅ (CrewRace) | 🟡 (Technical partial due to seed) |
| **Crew seed** | ✅ (validated hex) | ✅ | ❌ |
| **Home system seed** | ✅ (validated hex) | ✅ | ✅ |
| **Inventory editing** | ✅ (General, Tech, Cargo) | ✅ | 🟡 |
| **Inventory resize (W x H)** | ✅ (NUD controls) | ✅ | ✅ |
| **Base room detection** | ✅ (31 known room ObjectIDs) | ✅ (GetIsRoomInstalled) | ❌ |

## Frigate Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Frigate name** | ✅ | ✅ | ✅ |
| **Frigate type** | ✅ (with auto trait adjustment) | ✅ (with trait adjustment) | 🟡 |
| **Frigate race** | ✅ | ✅ | ✅ |
| **Home seed / Model seed** | ✅ (validated hex) | ✅ | ✅ |
| **Stats (0-10)** | ✅ | ✅ | ✅ |
| **Traits (5 slots)** | ✅ (empty = "^" marker) | ✅ (empty = "^") | 🟡 |
| **Class computation** | ✅ (net scoring: +1 beneficial, -1 negative) | ✅ (same algorithm) | ❌ |
| **Damage taken / Repair** | ✅ | ✅ | 🟡 |
| **Expedition counters** | ✅ | ✅ | 🟡 |
| **Expedition start time** | ✅ (DateTimePicker) | ✅ | ❌ |
| **Max frigates** | ✅ 30 | ✅ 30 | ✅ 30 |
| **Export/import individual frigate** | ✅ | 🟡 | ❌ |

### Frigate Type-Specific Auto-Adjustment

| Type | Auto Traits | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|---|
| **Normandy** | Diplomatic preset | ✅ | ✅ | ✅ |
| **DeepSpace** | Combat preset | ✅ | ✅ | ✅ |
| **GhostShip** | Exploration preset | ✅ | ✅ | ✅ |
| **Standard types** | No auto-adjustment | ✅ | ✅ | ✅ |

## Squadron Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Pilot race** | ✅ | ✅ | ✅ |
| **Pilot rank** | ✅ | ✅ | 🟡 |
| **NPC seed** | ✅ | ✅ | ✅ |
| **Ship seed** | ✅ | ✅ | ✅ |
| **Traits seed** | ✅ | ✅ | ❌ |
| **Ship type** | ✅ | ✅ | ✅ |
| **Has pilot toggle** | ✅ | ✅ | ❌ |
| **Is unlocked toggle** | ✅ | ✅ | ✅ |
| **Delete / invalidate** | ✅ | ✅ | ❌ |
| **Export/import** | ✅ | 🟡 | ❌ |
| **Max squadron slots** | ✅ 4 | ✅ 4 | ✅ 4 |

## Exocraft / Vehicle Editing

| Feature | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Vehicle inventory editing** | ✅ | ✅ | 🟡 |
| **Primary vehicle toggle** | ✅ | ✅ | ❌ |
| **Is deployed indicator** | ✅ | ✅ | ❌ |
| **Undeploy action** | ✅ | ✅ | ❌ |
| **Supercharge disabled** | ✅ (correct per game design) | ✅ | ❌ |

### Vehicle Types

| Type | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Roamer / Nomad / Colossus / Pilgrim** | ✅ | ✅ | 🟡 |
| **Submarine (Nautilon)** | ✅ | ✅ | 🟡 |
| **Mech (Minotaur)** | ✅ | ✅ | 🟡 |
| **Truck** | ✅ | ✅ | 🟡 |
| **Skiff** | ✅ (via Storage) | ❌ | 🟡 (via  Vehicle) |
| **Unknown vehicles** | ✅ | ✅ | ❌ |

## Seed Validation

All three editors handle seed values differently:

| Aspect | NMSE | NomNom | NMSSaveEditor |
|---|---|---|---|
| **Validation** | SeedHelper.NormalizeSeed (Trim, Upper, 0x normalize, hex check) | Pattern validation | Basic validation |
| **Scope** | All panels (ship, tool, freighter, frigate, squadron, companion, base NPC) | Per-wrapper | Limited |
| **Invalid seed handling** | Rejects non-hex input | Rejects non-hex | Accepts anything |
