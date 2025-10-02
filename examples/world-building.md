# World Building Examples

**Innovation Attribution**: The world building patterns documented here were pioneered by **Lost Mountain Games** (2025) to demonstrate sentient locations and unlimited spatial composition.

## Basic Location

### Simple Town

**Concept**: Standard location with connections and contents.

```javascript
entity Iron_Forge {
  // Identity
  plug.Name("Iron Forge")
  plug.Description("A mountain stronghold of the Iron Wolves clan")
  plug.Tag([location_tag, settlement_tag, fortress_tag])

  // Location Type
  plug.LocationType("Settlement")
  plug.LocationSubtype("Fortress")
  plug.Size("Medium")
  plug.Population(500)

  // Connections (neighboring locations)
  plug.Connections([
    mountain_pass_north,
    valley_road_south,
    forest_trail_east
  ])

  // Contents (what's inside this location)
  plug.Contains([
    great_hall_iron_wolves,
    merchants_quarter,
    smithy_district,
    barracks,
    armory_chamber
  ])

  // NPCs Present
  plug.NPCsPresent([
    war_chief_ironwind,
    thane_the_smith,
    merchant_gilda,
    guard_captain_stone
  ])

  // Environment
  plug.Climate("Temperate Mountain")
  plug.Weather("Clear")
  plug.Temperature(65)  // Fahrenheit

  // Game Mechanics
  plug.SafeZone(true)
  plug.RestingAllowed(true)
  plug.FastTravelPoint(true)
}
```

**MongoDB Document**:
```json
{
  "_id": "iron_forge_uuid",
  "plug": {
    "Name": "Iron Forge",
    "LocationType": "Settlement",
    "Connections": ["mountain_pass_north_uuid", "valley_road_south_uuid"],
    "Contains": ["great_hall_uuid", "merchants_quarter_uuid"],
    "NPCsPresent": ["war_chief_ironwind_uuid", "thane_the_smith_uuid"],
    "SafeZone": true
  }
}
```

## Hierarchical Locations

### Nested Location Structure

**Concept**: Locations within locations (rooms in building, buildings in town).

```javascript
// Top Level: Region
entity Northern_Mountains {
  plug.Name("Northern Mountains")
  plug.Tag([location_tag, region_tag])
  plug.LocationType("Region")

  plug.Contains([
    iron_forge,
    mountain_pass,
    hidden_valley,
    dragon_peak
  ])
}

// Second Level: Settlement
entity Iron_Forge {
  plug.Name("Iron Forge")
  plug.Tag([location_tag, settlement_tag])
  plug.LocationType("Settlement")

  // Parent location
  plug.Location(northern_mountains)

  // Child locations
  plug.Contains([
    great_hall,
    merchants_quarter,
    smithy_district
  ])
}

// Third Level: Building
entity Great_Hall {
  plug.Name("Great Hall of the Iron Wolves")
  plug.Tag([location_tag, building_tag])
  plug.LocationType("Building")

  // Parent location
  plug.Location(iron_forge)

  // Rooms inside building
  plug.Contains([
    throne_room,
    feast_hall,
    chieftains_chamber
  ])
}

// Fourth Level: Room
entity Throne_Room {
  plug.Name("Throne Room")
  plug.Tag([location_tag, room_tag])
  plug.LocationType("Room")

  // Parent location
  plug.Location(great_hall)

  // Contents of room
  plug.Contains([
    iron_throne,
    banner_iron_wolves,
    war_chief_ironwind  // NPC
  ])

  plug.Description("A grand hall with ancient banners and a throne of iron")
}
```

**Hierarchy**:
```
Northern Mountains (Region)
└── Iron Forge (Settlement)
    └── Great Hall (Building)
        └── Throne Room (Room)
            └── Iron Throne (Object)
```

**Pattern**: Each level uses `plug.Location()` to specify parent, and `plug.Contains()` to specify children. This enables:
- Breadcrumb navigation (Region > Settlement > Building > Room)
- "Where is X?" queries traverse hierarchy
- Automatic map generation from relationships

## Sentient Location

### Living Castle That Can Move

**Concept**: Location that is also a character with personality and goals.

```javascript
entity Wandering_Citadel {
  // LOCATION PLUGS
  plug.Name("The Wandering Citadel")
  plug.Description("An ancient fortress that walks on massive stone legs")
  plug.Tag([location_tag, building_tag, character_tag, legendary_tag, sentient_tag])

  // Location Properties
  plug.LocationType("Mobile Fortress")
  plug.Size("Massive")

  // Contains (rooms/areas inside)
  plug.Contains([
    citadel_entrance_hall,
    library_of_ages,
    throne_of_the_wanderer,
    observation_deck
  ])

  // Connections change as it moves
  plug.Connections([
    current_region  // Dynamically updated
  ])

  // CHARACTER PLUGS (sentient location)
  plug.Personality("Ancient, mysterious, speaks in riddles")
  plug.Will(20)         // Immensely powerful
  plug.Intelligence(18)
  plug.Alignment("True Neutral")

  // Can communicate
  plug.CanSpeak(true)
  plug.Languages(["Ancient Tongue", "Common", "All Known Languages"])
  plug.Dialogue([
    greeting_visitors,
    cryptic_warnings,
    historical_knowledge,
    movement_explanations
  ])

  // Goals & Motivations
  plug.Goal("Wander the world, observing history unfold")
  plug.Memory("Witnessed rise and fall of civilizations")
  plug.Age(5000)        // Years old

  // MOBILE LOCATION MECHANICS
  plug.CanMove(true)
  plug.MovementSpeed("5 miles per day")
  plug.CurrentLocation(plains_of_valor)
  plug.MovementPattern("Follows ancient ley lines")

  // Special Abilities
  plug.SpecialAbility([
    teleport_to_ley_line_nexus,
    phase_through_mountains,
    create_temporary_portal
  ])

  // Relationship with Visitors
  plug.Relationship(player_character, "Curious Observer", 50)
  plug.WillTransport("Only the worthy")

  // Game Mechanics
  plug.FastTravelHub(true)  // Can teleport to ley lines
  plug.SafeZone(true)
  plug.QuestGiver([
    quest_ancient_knowledge,
    quest_ley_line_corruption
  ])
}
```

**Innovation**: Location you can TALK to. Castle might refuse entry, negotiate for transport, or share ancient wisdom. Moves across the world creating dynamic fast-travel network.

## Dynamic Environment Location

### Forest with Time and Weather Systems

**Concept**: Location with dynamic state that changes over time.

```javascript
entity Moonwood_Forest {
  // Identity
  plug.Name("Moonwood Forest")
  plug.Description("An ancient elven forest that glows under moonlight")
  plug.Tag([location_tag, wilderness_tag, forest_tag, magical_tag])

  // Location Properties
  plug.LocationType("Wilderness")
  plug.LocationSubtype("Ancient Forest")
  plug.Size("Large")
  plug.Terrain("Dense Forest")

  // Connections
  plug.Connections([
    iron_forge,
    elven_outpost,
    shadow_grove,
    moonlit_clearing
  ])

  // Contents
  plug.Contains([
    ancient_oak_grove,
    hunters_camp,
    hidden_spring,
    ruins_of_old_shrine
  ])

  // Dynamic Environment
  plug.Weather("Clear Night")
  plug.WeatherSystem([
    clear_nights,
    misty_mornings,
    rain_showers,
    thunderstorms
  ])

  // Time-Based States
  plug.DayAppearance("Dark, mysterious, animals quiet")
  plug.NightAppearance("Glowing with ethereal moonlight, magical energy visible")

  // Time-Dependent Properties
  plug.DangerLevel({
    day: "Low",
    night: "Medium",
    full_moon: "High"      // Werewolves active
  })

  // Environmental Hazards
  plug.Hazards([
    wild_animals,
    poisonous_plants,
    unstable_magic,
    werewolf_packs  // Only at night
  ])

  // Resources
  plug.Resources([
    moonwood_lumber,
    magical_herbs,
    moonstone_crystals,
    fresh_water
  ])

  // NPCs (change by time)
  plug.NPCsPresent({
    day: [
      forest_ranger,
      herb_gatherer,
      lost_traveler
    ],
    night: [
      elven_night_patrol,
      werewolf_pack,
      moon_priestess
    ]
  })

  // Environmental Effects
  plug.AmbientMagic("Lunar energy strengthens nature magic")
  plug.BuffsApplied([
    moonlight_blessing,  // Bonus to nature magic
    forest_stealth      // Easier to hide
  ])

  // Game Mechanics
  plug.RestingAllowed(true)
  plug.CampingRisk("Medium")  // Random encounters
  plug.HarvestingEnabled(true)
}
```

**Pattern**: Location state changes based on time, weather, and moon phase. Different NPCs, hazards, and opportunities at different times.

## Interconnected Dungeon

### Multi-Level Dungeon with Shortcuts

**Concept**: Complex dungeon with multiple paths and unlockable shortcuts.

```javascript
entity Ancient_Catacombs {
  // Identity
  plug.Name("Ancient Catacombs")
  plug.Description("Buried crypts beneath the old city, filled with undead")
  plug.Tag([location_tag, dungeon_tag, underground_tag, dangerous_tag])

  // Dungeon Properties
  plug.LocationType("Dungeon")
  plug.DungeonType("Catacombs")
  plug.DungeonLevel(5)  // Floors deep
  plug.DifficultyRating("Hard")
  plug.RecommendedLevel(15)

  // Dungeon Structure
  plug.Contains([
    entrance_chamber,
    burial_halls_upper,
    burial_halls_lower,
    crypt_of_kings,
    forbidden_sanctum  // Boss room
  ])

  // Connections (entrances/exits)
  plug.Connections([
    old_city_sewers,      // Main entrance
    secret_passage_cathedral  // Hidden exit
  ])

  // Environmental Conditions
  plug.Lighting("Dark")
  plug.Atmosphere("Oppressive, thick with death")
  plug.Temperature(50)   // Cold
  plug.Hazards([
    unstable_floors,
    poison_gas_traps,
    cursed_areas
  ])

  // Enemies
  plug.Enemies([
    skeleton_warriors,
    zombie_hordes,
    wraith_sentinels,
    lich_lord  // Boss
  ])

  // Loot & Rewards
  plug.Treasure([
    ancient_artifacts,
    gold_hoards,
    legendary_weapons,
    spell_tomes
  ])

  // Dungeon Mechanics
  plug.Shortcuts([
    elevator_to_lower_levels,  // Unlocks after boss
    secret_passage_to_surface  // Requires key
  ])

  plug.Checkpoints([
    safe_room_level_2,
    ancient_shrine_level_4
  ])

  plug.BossRoom(forbidden_sanctum)
  plug.BossDefeated(false)  // State tracking

  // Respawn Mechanics
  plug.EnemyRespawn("Daily")
  plug.LootRespawn("Weekly")

  // Quest Integration
  plug.RelatedQuests([
    quest_cleanse_catacombs,
    quest_retrieve_artifact,
    quest_ancient_evil
  ])
}
```

**Pattern**: Dungeon as interconnected location graph. Shortcuts unlock after boss defeats. State tracking for cleared areas.

## Living World Region

### Region with Faction Control and War

**Concept**: Large region with dynamic faction control and territory battles.

```javascript
entity Borderlands {
  // Identity
  plug.Name("The Borderlands")
  plug.Description("Contested territory between three kingdoms")
  plug.Tag([location_tag, region_tag, warzone_tag])

  // Region Properties
  plug.LocationType("Region")
  plug.Size("Massive")
  plug.Terrain("Mixed - plains, forests, hills")

  // Contains (major locations)
  plug.Contains([
    fort_valor,
    contested_village,
    no_mans_land,
    ancient_battlefield
  ])

  // Faction Control (dynamic)
  plug.ControlledBy({
    fort_valor: kingdom_of_valor,
    contested_village: null,        // Neutral/contested
    no_mans_land: null
  })

  // War State
  plug.WarStatus("Active Conflict")
  plug.Factions([
    kingdom_of_valor,
    shadow_empire,
    iron_wolves_clan
  ])

  // Faction Territories
  plug.FactionTerritory({
    kingdom_of_valor: ["fort_valor", "eastern_outpost"],
    shadow_empire: ["dark_tower", "shadow_camp"],
    iron_wolves_clan: ["wolf_den", "northern_border"]
  })

  // Dynamic Events
  plug.ActiveEvents([
    siege_of_fort_valor,
    patrol_skirmishes,
    refugee_caravans
  ])

  // Danger Levels by Area
  plug.DangerMap({
    fort_valor: "Low",            // Safe zone
    contested_village: "High",    // Active fighting
    no_mans_land: "Extreme"       // Constant combat
  })

  // NPCs (faction-dependent)
  plug.NPCsByFaction({
    kingdom_of_valor: [captain_valor, soldiers, refugees],
    shadow_empire: [dark_commander, shadow_troops],
    iron_wolves_clan: [war_chief_ironwind, warriors]
  })

  // Reputation Effects
  plug.ReputationZones(true)  // Reputation affects NPC behavior

  // Quests (faction-specific)
  plug.QuestsByFaction({
    kingdom_of_valor: [quest_defend_fort, quest_rescue_refugees],
    shadow_empire: [quest_sabotage, quest_assassination],
    iron_wolves_clan: [quest_secure_border, quest_peace_treaty]
  })

  // State Changes
  plug.ControlChangeConditions([
    "Fort captured → Control shifts",
    "Treaty signed → Becomes peaceful",
    "Boss defeated → Faction collapses"
  ])
}
```

**Pattern**: Region state changes based on player actions and faction wars. Territory control affects available quests, NPC behavior, and danger levels.

## Cosmic Hierarchy Location

### Multiverse Location System

**Concept**: Nested cosmic hierarchy from multiverse to individual rooms.

```javascript
// Level 1: Multiverse
entity The_Multiverse {
  plug.Name("The Multiverse")
  plug.Tag([location_tag, cosmic_tag])
  plug.LocationType("Cosmic")
  plug.Contains([
    material_plane,
    shadow_realm,
    celestial_sphere,
    abyssal_void
  ])
}

// Level 2: Plane of Existence
entity Material_Plane {
  plug.Name("Material Plane")
  plug.Tag([location_tag, plane_tag])
  plug.LocationType("Plane")
  plug.Location(the_multiverse)
  plug.Contains([
    world_reavers,
    world_forgotten,
    world_shattered
  ])
}

// Level 3: World
entity World_Reavers {
  plug.Name("Reavers Universe")
  plug.Tag([location_tag, world_tag])
  plug.LocationType("World")
  plug.Location(material_plane)
  plug.Contains([
    continent_valoria,
    continent_shadows,
    ocean_depths
  ])
}

// Level 4: Continent
entity Continent_Valoria {
  plug.Name("Valoria")
  plug.Tag([location_tag, continent_tag])
  plug.LocationType("Continent")
  plug.Location(world_reavers)
  plug.Contains([
    northern_mountains,
    central_plains,
    southern_forests
  ])
}

// Level 5: Region
entity Northern_Mountains {
  plug.Name("Northern Mountains")
  plug.Tag([location_tag, region_tag])
  plug.LocationType("Region")
  plug.Location(continent_valoria)
  plug.Contains([
    iron_forge,
    dragon_peak,
    hidden_valley
  ])
}

// Level 6: Settlement (continues as before...)
```

**Hierarchy**:
```
Multiverse
└── Material Plane
    └── World: Reavers
        └── Continent: Valoria
            └── Region: Northern Mountains
                └── Settlement: Iron Forge
                    └── Building: Great Hall
                        └── Room: Throne Room
                            └── Object: Iron Throne
```

**Pattern**: Infinite nesting depth. Each level uses same plug structure. Enables cosmic-scale and room-scale locations in same system.

## Attribution

The world building patterns documented here were pioneered by **Lost Mountain Games** (2025), demonstrating sentient locations, mobile fortresses, dynamic environments, and unlimited spatial hierarchy through pluggable composition.

**Innovation**: Locations as characters (talking castles), dynamic state management (time/weather/faction control), and cosmic hierarchy (multiverse to room-scale) using unified plug system.

---

**Copyright © 2025 Lost Mountain Games**
