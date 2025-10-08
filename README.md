# Plug DSL: Schema-Driven Game Development

A data-driven approach to game entity design that eliminates architectural brittleness through composable plug systems.

## Core Philosophy

**Entities are collections of plugs, not instances of classes.**

Traditional game development often relies on rigid class hierarchies that become brittle as requirements evolve. Plug DSL treats entities as flexible collections of data plugs, enabling fluid composition without architectural constraints and limiting data to necessary fields.

## Foundational Elements

### plug.Name() - Human-Readable Identity System

The Name plug establishes the dual-identity system that enables both human authoring and system stability:

```c
// kael_ironwind.entity
plug.Name("Kael Ironwind");
plug.Description(Grizzled Warrior Description);  // Referenced entity - IDE highlights as link
plug.Age(34);
plug.Will(12);

// Grizzled Warrior Description.entity
plug.Name("Grizzled Warrior Description");
plug.Text("A grizzled warrior bearing the scars of countless battles");
```

**Dual Syntax Pattern:**
- **Literal Values**: `plug.Description("Direct text string")`
- **Referenced Entities**: `plug.Description(Grizzled Warrior Description)` - IDE syntax highlighting indicates link

**Key Properties:**
- **Human-Readable Authoring**: Developers write meaningful names for all entities
- **UUID Storage**: System maintains permanent references via UUID for all linked content
- **Dynamic Updates**: Entity names can change without breaking references
- **VS Code Integration**: Referenced entities become navigable links with hover previews
- **Content Reusability**: Entities like character class can be referenced across multiple characters
- **Offline Compatibility**: UUID-based storage enables distributed development

### plug.Tag() - Modular Organization System

Tags provide flexible categorization and IDE enhancement through structured metadata:

```c
// character_tag.entity
plug.Name("Character");
plug.Description(Entites Representing Player Characters and NPCs);
plug.Color("#4A90E2");
plug.Icon("person");
plug.HintStructure("Will, Might, Endurance, Speed");

// warrior_tag.entity
plug.Name("Warrior");
plug.Description(A Combat Focused Character Specialization);
plug.Color("#8B4513");
plug.Icon("sword");
plug.HintStructure("CombatTechniques, Equipment, Reputation");

// Apply multiple tags to entities
plug.Name("Kael Ironwind");
plug.Tag(character_tag, warrior_tag);
```

**Tag Structure Benefits:**
- **Hint Floats**: IDE displays each tag's structure and expected plugs
- **Contextual Assistance**: VS Code shows relevant plug suggestions based on applied tags
- **Visual Organization**: Color and icon coding for immediate entity type recognition
- **Modular Composition**: Multiple tags per entity enable complex categorization
- **Extensible Metadata**: Tags themselves are entities that can contain any plug combination

## Basic Syntax

```c
// Complete entity example
plug.Name("Kael Ironwind");
plug.Tag(character_tag, warrior_tag);
plug.Description(A grizzled warrior bearing the scars of countless battles);
plug.Age(34);
plug.Will(12);
plug.Might(16);
plug.Endurance(14);
plug.Speed(10);
```

Every entity is a collection of plugs. No predefined types, no class inheritance.

## Entity References with UUID Resolution

### Human-Friendly Authoring

```c
// Author writes readable names for entity references
plug.Chieftain(War Chief Ironwind);
plug.Equipment(Ironwolf Axe, Clan Banner);
plug.Description(Grizzled Warrior Description);

// Or uses literal values for simple data
plug.Age(34);
plug.Will("Determined and unwavering");
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

## MongoDB Storage

Entities transpose 1-to-1 to JSON, creating flat MongoDB documents that are transparent to LLMs and optimized for querying:

```javascript
// kael_ironwind.entity storage
{
  "_id": "550e8400-e29b-41d4-a716-446655440000",
  "Name": "Kael Ironwind",
  "Description": "f47ac10b-58cc-4372-a567-0e02b2c3d479", // UUID reference
  "Age": 34,                                              // Literal value
  "Will": 16,
  "Equipment": [
    "7c9e6679-7425-40de-944b-e07fc1f90ae7"
  ]
}

// Grizzled Warrior Description.entity storage
{
  "_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "Name": "Grizzled Warrior Description",
  "Text": "A grizzled warrior bearing the scars of countless battles"
}
```

**1-to-1 Transposition Benefits:**
- **LLM Comprehension**: AI systems can directly read and modify entity data without translation layers
- **MongoDB Optimization**: Native JSON storage enables efficient indexing and querying
- **No Serialization Overhead**: Direct mapping eliminates conversion performance costs
- **Transparent Debugging**: Stored data matches authored structure exactly

## Key Benefits

**Eliminates Architectural Brittleness**: Add new features through plug composition, not system modification.

**LLM-Friendly**: Flat document structure is transparent to AI systems for content generation and modification.

**Import Safety**: UUID-based references prevent conflicts when sharing content between projects.

**Future-Proof**: System grows through plug addition, not architectural changes.

## Extensible Composition

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

## Additional Examples

See [EXAMPLES.md](EXAMPLES.md) for cross-category entity examples, advanced composition patterns, and complex plug system implementations.

## Implementation Notes

**Performance**: Flat document structure enables efficient MongoDB indexing and caching strategies.

**Redis Integration**: UUID-based entity keys map naturally to Redis caching, enabling sub-millisecond entity resolution for frequently accessed game objects.

**Tooling**: IDE integration provides entity reference navigation, hover information panels, and syntax highlighting for enhanced developer experience.

**Storage**: Flat MongoDB document structure ensures LLM transparency and efficient querying.

## License

MIT License - see [LICENSE](LICENSE) for details.