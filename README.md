# Plug DSL: Schema-Driven Game Development

A data-driven approach to game entity design that eliminates architectural brittleness through composable plug systems.

## Core Philosophy

**Entities are collections of plugs, not instances of classes.**

Traditional game development relies on rigid class hierarchies that become brittle as requirements evolve. Plug DSL treats entities as flexible collections of data plugs, enabling unlimited composition without architectural constraints.

## Basic Syntax

```c
// kael_ironwind.entity
plug.Name("Kael Ironwind");
plug.Description("A grizzled warrior bearing the scars of countless battles");
plug.Age(34);
plug.Will(12);
plug.Might(16);
plug.Endurance(14);
plug.Speed(10);
```

Every entity is simply a collection of plugs. No predefined types, no class inheritance, no architectural limitations.

## Unlimited Composition

### Any Entity, Any Plugs

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

Want a talking location? Add character plugs. Want a mobile fortress? Add movement plugs. The system doesn't enforce artificial boundaries.

## Entity References with UUID Resolution

### Human-Friendly Authoring

```c
// Author writes readable names
plug.Chieftain(war_chief_ironwind);
plug.Equipment(ironwolf_axe, clan_banner);
```

### Stable UUID Storage

```javascript
// System stores UUIDs for stability
{
  "Chieftain": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "Equipment": [
    "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "123e4567-e89b-12d3-a456-426614174000"
  ]
}
```

### Automatic Name Resolution

When "War Chief Ironwind" becomes "Elder Ironwind", all UI displays update automatically while UUID references remain stable.

## Extensibility Through Plug Addition

### Core System

```c
// Ships with basic plugs
plug.Name(string);
plug.Description(text);
plug.Will(number);
plug.Location(reference);
```

### Namespace Extension

```c
// Import additional plug libraries
import DnD from "plugs/dnd_5e";
import Custom from "plugs/my_world";

// Use multiple plug systems
plug.Name("Kael Ironwind");     // Core
DnD.Strength(18);               // D&D system
Custom.Reputation(Clan, High);  // Custom system
```

### Custom Plug Definition

```c
// Define new plug types
define_plug MagicAffinity(aspect: enum, strength: number) {
    validate(aspect, ["Fire", "Water", "Earth", "Air"]);
    validate(strength, {min: 1, max: 10});
}

// Use immediately
plug.MagicAffinity(Fire, 8);
plug.MagicAffinity(Earth, 3);
```

## MongoDB Storage

Entities are stored as flat MongoDB documents, making them transparent to LLMs and easy to query:

```javascript
{
  "_id": "550e8400-e29b-41d4-a716-446655440000",
  "Name": "Kael Ironwind",
  "Will": 16,
  "Might": 18,
  "Equipment": [
    "7c9e6679-7425-40de-944b-e07fc1f90ae7"
  ],
  "MagicAffinity": [
    {"aspect": "Fire", "strength": 8},
    {"aspect": "Earth", "strength": 3}
  ]
}
```

## Key Benefits

**Eliminates Architectural Brittleness**: Add new features through plug composition, not system modification.

**Unlimited Creativity**: Any entity can have any combination of plugs - characters can be locations, items can be sentient, locations can move.

**LLM-Friendly**: Flat document structure is transparent to AI systems for content generation and modification.

**Import Safety**: UUID-based references prevent conflicts when sharing content between projects.

**Future-Proof**: System grows through plug addition, not architectural changes.

## Additional Examples

See [EXAMPLES.md](EXAMPLES.md) for more detailed examples of cross-category entities, custom plug systems, and advanced composition patterns.

## Implementation Notes

This repository documents the conceptual patterns developed for the Reavers Engine. The concepts are freely available under MIT license for community implementation and extension.

**Performance**: The Reavers Engine implementation achieves sub-25ms entity resolution through MongoDB indexing and caching strategies.

**Tooling**: IDE integration provides entity reference navigation, hover information panels, and syntax highlighting for enhanced developer experience.

## License

MIT License - see [LICENSE](LICENSE) for details.

---

**Developed by Lost Mountain Games (2025)**  
Concepts documented for community benefit and prior art establishment.