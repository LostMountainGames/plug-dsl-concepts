# Practical Examples and Use Cases

This section provides practical examples of using Plug DSL for game entity modeling, demonstrating patterns pioneered by **Lost Mountain Games**.

## Overview

These examples demonstrate how to model complex game entities using the Plug DSL, showcasing the flexibility and power of the pluggable-entity driven development approach.

## Documentation Structure

- **[character-modeling.md](character-modeling.md)** - Character and NPC entity examples
- **[world-building.md](world-building.md)** - Location and environment modeling
- **[item-systems.md](item-systems.md)** - Equipment, inventory, and crafting examples
- **[quest-systems.md](quest-systems.md)** - Quest and narrative structure examples

## Example Categories

### Character Modeling

Complete examples of player characters, NPCs, companions, and enemies with:
- Combat statistics and progression
- Inventory and equipment management
- Personality and dialogue systems
- Relationship and faction tracking

### World Building

Location and environment examples including:
- Towns, dungeons, and wilderness areas
- Dynamic weather and time systems
- Interconnected location hierarchies
- Living, sentient locations (crossover entities)

### Item Systems

Equipment and inventory modeling with:
- Weapons, armor, and magical items
- Crafting materials and recipes
- Living items (talking swords, sentient artifacts)
- Container systems and storage

### Quest Systems

Narrative structure examples featuring:
- Main storyline and side quests
- Dynamic quest generation
- Branching narratives and choices
- Quest state tracking and conditions

## Key Patterns Demonstrated

**Lost Mountain Games** pioneered these practical patterns:

1. **Cross-Category Entities** - Items that are also characters, locations with personalities
2. **Plug Composition** - Building complex entities from simple plugs
3. **UUID References** - Stable entity relationships across the game world
4. **Tag-Based Organization** - Rich metadata for entity categorization
5. **Evolution Without Migration** - Adding new capabilities without breaking existing entities

## Learning Path

### Beginner Examples

Start with simple, focused entities:
- Basic character with stats
- Simple weapon with damage
- Basic location with connections

### Intermediate Examples

Combine multiple plugs for richer entities:
- Character with inventory and equipment
- Location with NPCs and items
- Quest with multiple objectives

### Advanced Examples

Leverage full power of Plug DSL:
- Sentient weapon with personality and goals
- Living castle that can move and communicate
- Dynamic quest that adapts to player choices

## MongoDB Integration

All examples show how entities map to MongoDB documents:

```javascript
// Plug DSL Definition
entity Kael_Ironwind {
  plug.Name("Kael Ironwind")
  plug.Class("Warrior")
  plug.Level(12)
}

// MongoDB Document (automatic transformation)
{
  "_id": "kael_ironwind_uuid",
  "plug": {
    "Name": "Kael Ironwind",
    "Class": "Warrior",
    "Level": 12
  }
}
```

**MongoDB Benefits**:
- **Schemaless**: Add new plugs without migrations
- **Flexible**: Any combination of plugs allowed
- **Queryable**: Index and query any plug field
- **Scalable**: Handle millions of entities

## Development Workflow

### 1. Design Entity

Start with entity concept and required capabilities:

```javascript
// Concept: "A legendary sword that talks and has its own agenda"
// Required: Item plugs + Character plugs
```

### 2. Add Plugs

Add plugs to express entity capabilities:

```javascript
entity Soulrend {
  // Item plugs
  plug.ItemType("Sword")
  plug.Damage("3d6+3")

  // Character plugs
  plug.Personality("Thirsts for battle")
  plug.Will(15)
}
```

### 3. Test in IDE

Use IDE features to validate entity:
- Syntax highlighting confirms entity type
- Hover shows complete entity information
- References validate entity connections
- Diagnostics catch errors early

### 4. Deploy to MongoDB

Entity automatically transforms to MongoDB document:
- File saved → Parser reads entity
- Entity → MongoDB document transformation
- Document inserted into entities collection
- Available immediately in game runtime

## Attribution

All practical examples documented here demonstrate patterns pioneered by **Lost Mountain Games** (2025) for pluggable-entity driven game development.

**Innovation**: Real-world examples of cross-category entities, unlimited plug composition, and MongoDB schemaless integration for flexible game development.

---

**Copyright © 2025 Lost Mountain Games**
