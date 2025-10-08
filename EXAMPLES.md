# Plug DSL Examples

Practical examples demonstrating the flexibility of plug-based entity composition.

## Unlimited Composition Patterns

### Any Entity, Any Plugs

The system doesn't enforce artificial boundaries between entity types:

```c
// character.entity
plug.Name("Kael Ironwind");
plug.Will(16);
plug.Personality("Gruff but loyal");

// location.entity  
plug.Name("Whispering Grotto");
plug.LocationType(AncientRuin);
plug.Region(NorthernWildlands);

// hybrid.entity (location + character)
plug.Name("Heart of the Worldtree");
plug.LocationType(CosmicNexus);    // Location behavior
plug.Will(20);                     // Character behavior
plug.Personality("Ancient wisdom"); // Character behavior
```

Want a talking location? Add character plugs. Want a mobile fortress? Add movement plugs.

## Cross-Category Entities

### Sentient Location

```c
// worldtree_heart.entity
plug.Name("Heart of the Worldtree");
plug.Description("The ancient core where all planar strings converge");
plug.LocationType(CosmicNexus);
plug.MagicBonus(All, +3);
plug.Personality("Ancient, wise, speaks in riddles of cosmic truth");
plug.Will(20);
plug.Languages(PrimordialTongue, AllPlanarDialects);
```

### Living Weapon

```c
// soulrend_glaive.entity
plug.Name("Soulrend Glaive");
plug.ItemType(MagicalGlaive);
plug.Damage("3d6+3");
plug.Personality("Thirsts for battle, whispers of ancient victories");
plug.Will(15);
plug.CanCommunicate(Telepathic);
plug.Memories("Recalls every soul it has claimed");
```

### Mobile Fortress

```c
// ironwolf_fortress.entity
plug.Name("Ironwolf Mobile Fortress");
plug.LocationType(MobileFortress);
plug.Region(NorthernWildlands);
plug.Contains(great_hall, armory, barracks);
plug.Speed(2);
plug.MovementType(Wheeled);
plug.Population(500);
plug.Chieftain(war_chief_ironwind);
```

## Extensible Plug Systems

### Game System Integration

```c
// hybrid_character.entity
import Reavers from "plugs/reavers_universe";
import DnD from "plugs/dnd_5e";

Reavers.Name("Kael Ironwind");
Reavers.Will(16);
DnD.Strength(18);
DnD.Class(Fighter);
DnD.Level(5);
```

### Custom Plug Definition

```c
// Define reputation system
define_plug Reputation(faction: reference, level: enum) {
    validate(faction, {type: "entity_reference"});
    validate(level, ["Hated", "Disliked", "Neutral", "Liked", "Revered"]);
    store("reputations", {faction: faction, level: level}, mode: "append");
}

// Use with any entity
plug.Reputation(iron_wolves_clan, Revered);
plug.Reputation(southern_kingdoms, Hated);
plug.Reputation(whispering_grotto, Curious);  // Even locations!
```

## Organizational Tags

```c
// character_tag.entity
plug.Name("Character");
plug.Description("Entities representing people and NPCs");
plug.Color("#4A90E2");
plug.Icon("person");

// weapon_tag.entity
plug.Name("Weapon");
plug.Description("Items used for combat");
plug.Color("#E94B3C");
plug.Icon("sword");

// Use tags for organization
// kael_ironwind.entity
plug.Name("Kael Ironwind");
plug.Tag(character_tag);
plug.Tag(warrior_tag);
plug.Tag(ironwolves_tag);
```

## Reference Resolution

### Authoring with Names

```c
// Author writes readable references
plug.Chieftain(war_chief_ironwind);
plug.Equipment(ironwolf_axe, clan_banner, war_horn);
plug.Location(ironwolf_stronghold);
```

### Storage with UUIDs

```javascript
// System stores stable UUIDs
{
  "Chieftain": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "Equipment": [
    "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "123e4567-e89b-12d3-a456-426614174000",
    "456e7890-f123-45ab-cdef-567890123456"
  ],
  "Location": "789f0123-4567-89ab-cdef-012345678901"
}
```

### Display with Current Names

When "War Chief Ironwind" is renamed to "Elder Ironwind", all UI displays update automatically while UUID references remain stable.

## Complex Data Types

```c
// Multi-value plugs
plug.MagicAffinity(Fire, 8);
plug.MagicAffinity(Earth, 3);

// Structured data
plug.CombatTechniques(BasicStrike:Tier3, Charge:Tier2, Guard:Tier1);
plug.WordsOfPower(Firebolt:Tier2, StoneWall:Tier1);
plug.Reputation(IronWolves:Revered, Enemies:Feared);

// Validation
plug.Will(16);  // Validates range 3-18
plug.Age(34);   // Validates positive number
plug.Clan(IronWolves);  // Validates against enum
```

These examples demonstrate how plug composition eliminates traditional entity boundaries while maintaining type safety and referential integrity.