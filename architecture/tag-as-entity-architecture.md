# Tag-as-Entity Architecture

**Innovation Attribution**: The tag-as-entity pattern was pioneered by **Lost Mountain Games** (2025), transforming simple string tags into rich, evolvable entities with metadata and behavior.

## Core Innovation

Traditional game development uses tags as simple strings or enums:
```javascript
entity.tags = ["character", "warrior", "ironwolves"]  // Just strings
```

**Lost Mountain Games Innovation**: Tags are themselves entities with rich metadata, enabling hierarchical organization, visual presentation, and evolving categorization without rigid schemas.

## Tags Are Entities

### Tag Entity Structure

Tags are full entities stored as `.entity` files with their own plugs:

```javascript
// character_tag.entity
entity character_tag {
  plug.Name("Character")
  plug.Description("Entities representing people, NPCs, and player characters")
  plug.Color("#4A90E2")              // UI display color
  plug.Icon("person")                // UI icon
  plug.SortOrder(1)                  // Display ordering
  plug.Category("EntityType")        // Tag category grouping
  plug.Tag(entity_type_tag)          // Tags can have tags!
}

// warrior_tag.entity
entity warrior_tag {
  plug.Name("Warrior")
  plug.Description("Combat-focused characters skilled in melee warfare")
  plug.Color("#E94B3C")
  plug.Icon("sword")
  plug.SortOrder(5)
  plug.Category("CharacterClass")
  plug.Tag(character_tag)            // Hierarchical - warrior is a type of character
  plug.RequiredPlugs(["Might", "Endurance"])  // Validation metadata
}

// magical_tag.entity
entity magical_tag {
  plug.Name("Magical")
  plug.Description("Entities with supernatural or mystical properties")
  plug.Color("#9B59B6")
  plug.Icon("sparkles")
  plug.Rarity("Uncommon")
  plug.GlowEffect(true)              // Visual effect metadata
  plug.Tag(property_tag)
}
```

**Key Insight**: Tags have the same structure as any other entity. They're not special—they're just entities used for organization.

### Using Tag Entities

Entities reference tag entities like any other entity reference:

```javascript
// kael_ironwind.entity
entity Kael_Ironwind {
  plug.Name("Kael Ironwind")
  plug.Description("A grizzled warrior")
  plug.Will(12)
  plug.Might(16)

  // Tag references (UUID-based like all references)
  plug.Tag(character_tag)
  plug.Tag(warrior_tag)
  plug.Tag(ironwolves_tag)
  plug.Tag(veteran_tag)

  // Multiple tags provide rich categorization
}

// Stored in MongoDB with UUID references
{
  "_id": "kael_uuid",
  "plug": {
    "Name": "Kael Ironwind",
    "Tag": [
      "character_tag_uuid",
      "warrior_tag_uuid",
      "ironwolves_tag_uuid",
      "veteran_tag_uuid"
    ]
  }
}
```

## Rich Metadata Benefits

### Visual Presentation

Tag metadata drives UI presentation:

```javascript
// UI can render entities based on tag metadata
function renderEntity(entity) {
  const tags = entity.getPlugs("Tag").map(resolveTagEntity)

  // Get visual properties from primary tag
  const primaryTag = tags[0]
  const color = primaryTag.getPlug("Color")      // "#4A90E2"
  const icon = primaryTag.getPlug("Icon")        // "person"

  return `
    <div style="border-color: ${color}">
      <icon name="${icon}" />
      <span>${entity.getPlug("Name")}</span>
      ${tags.map(tag => renderTagBadge(tag))}
    </div>
  `
}

// Tags provide rich styling without hardcoded UI logic
```

### Hierarchical Organization

Tags can have parent tags, creating hierarchies:

```javascript
// Entity type hierarchy
entity_type_tag
  ├─ character_tag
  │   ├─ warrior_tag
  │   ├─ mage_tag
  │   └─ rogue_tag
  ├─ location_tag
  │   ├─ dungeon_tag
  │   └─ town_tag
  └─ item_tag
      ├─ weapon_tag
      └─ armor_tag

// Implementation through tag-has-tag references
entity warrior_tag {
  plug.Tag(character_tag)  // Warrior is a type of character
}

entity character_tag {
  plug.Tag(entity_type_tag)  // Character is a type of entity
}

// Query all characters (including warriors, mages, rogues)
db.entities.find({
  "plug.Tag": "character_tag_uuid"  // Direct tag
})
// Plus entities tagged with warrior_tag, mage_tag, etc.
```

### Validation Metadata

Tags can define requirements for entities that use them:

```javascript
// warrior_tag.entity
entity warrior_tag {
  plug.Name("Warrior")
  plug.RequiredPlugs(["Might", "Endurance"])  // Entities must have these
  plug.RecommendedPlugs(["WeaponSkill", "Armor"])
  plug.ConflictingTags(["mage_tag", "pacifist_tag"])  // Can't combine
}

// Validation system uses tag metadata
function validateEntity(entity) {
  const tags = entity.getPlugs("Tag").map(resolveTagEntity)

  for (const tag of tags) {
    // Check required plugs
    const required = tag.getPlug("RequiredPlugs") || []
    for (const plugName of required) {
      if (!entity.hasPlug(plugName)) {
        throw new Error(`Tag ${tag.Name} requires plug ${plugName}`)
      }
    }

    // Check conflicting tags
    const conflicts = tag.getPlug("ConflictingTags") || []
    for (const conflictTag of conflicts) {
      if (entity.hasTag(conflictTag)) {
        throw new Error(`Tags ${tag.Name} and ${conflictTag} conflict`)
      }
    }
  }
}
```

## Multiple Tag Categories

Entities can have tags from multiple categories:

```javascript
// moving_castle.entity
entity The_Moving_Castle {
  plug.Name("The Moving Castle")

  // Type tags
  plug.Tag(location_tag)      // It's a place you can visit
  plug.Tag(enemy_tag)         // It's also a boss enemy

  // Property tags
  plug.Tag(magical_tag)       // Has supernatural properties
  plug.Tag(boss_tag)          // Major encounter
  plug.Tag(mobile_tag)        // Can move

  // Faction tags
  plug.Tag(neutral_tag)       // Not aligned with any faction

  // Rarity tags
  plug.Tag(unique_tag)        // Only one exists

  // Multiple categories provide rich, multi-dimensional organization
}
```

### Tag Category Management

```javascript
// Define tag categories as entities
entity entity_type_tag {
  plug.Name("Entity Type")
  plug.Description("Categorizes what type of entity this is")
  plug.Color("#333333")
  plug.Exclusive(true)  // Entity can only have one tag from this category
}

entity property_tag {
  plug.Name("Property")
  plug.Description("Describes properties or characteristics")
  plug.Color("#666666")
  plug.Exclusive(false)  // Entity can have multiple property tags
}

// Validation enforces category rules
function validateTagCategories(entity) {
  const tags = entity.getPlugs("Tag").map(resolveTagEntity)
  const categoriesUsed = new Map()

  for (const tag of tags) {
    const category = tag.getPlug("Category")
    const categoryEntity = resolveEntity(category)

    if (categoryEntity.getPlug("Exclusive")) {
      if (categoriesUsed.has(category)) {
        throw new Error(`Can only have one tag from category ${category}`)
      }
      categoriesUsed.set(category, tag)
    }
  }
}
```

## Tag Evolution

### Tags Can Evolve

Because tags are entities, they can gain plugs over time:

```javascript
// Week 1: Simple tag
entity warrior_tag {
  plug.Name("Warrior")
  plug.Color("#E94B3C")
}

// Month 1: Add metadata
entity warrior_tag {
  plug.Name("Warrior")
  plug.Color("#E94B3C")
  plug.Icon("sword")                    // Added icon
  plug.Description("Combat specialists")  // Added description
}

// Month 6: Add validation rules
entity warrior_tag {
  plug.Name("Warrior")
  plug.Color("#E94B3C")
  plug.Icon("sword")
  plug.Description("Combat specialists")
  plug.RequiredPlugs(["Might", "Endurance"])  // Added requirements
  plug.BonusPlugs(["BattleCry", "Tactics"])    // Added bonus plugs
}

// Tags evolve with the game
// No migration needed - MongoDB schemaless
```

### Community Tag Extensions

Players/modders can create custom tag entities:

```javascript
// Custom mod tag
entity necromancer_tag {
  plug.Name("Necromancer")
  plug.Description("Dark magic specialist controlling undead")
  plug.Color("#2C003E")
  plug.Icon("skull")
  plug.Tag(mage_tag)                    // Extends official mage tag
  plug.RequiredPlugs(["DarkMagic", "UndeadControl"])
  plug.ModAuthor("PlayerName")
  plug.ModVersion("1.0.0")
}

// Mod entities use custom tags
entity Necromancer_Character {
  plug.Tag(character_tag)
  plug.Tag(necromancer_tag)  // Custom tag
  plug.DarkMagic(15)
  plug.UndeadControl(10)
}
```

## Query Patterns

### Find Entities by Tag

```javascript
// Find all warriors
db.entities.find({
  "plug.Tag": "warrior_tag_uuid"
})

// Find all magical warriors
db.entities.find({
  "plug.Tag": {
    $all: ["warrior_tag_uuid", "magical_tag_uuid"]
  }
})

// Find entities with any combat-related tag
const combatTags = ["warrior_tag_uuid", "archer_tag_uuid", "fighter_tag_uuid"]
db.entities.find({
  "plug.Tag": { $in: combatTags }
})
```

### Tag-Based Filtering

```javascript
// UI tag filter
function filterByTags(entities, selectedTags) {
  return entities.filter(entity => {
    const entityTags = entity.getPlugs("Tag")
    return selectedTags.every(tag => entityTags.includes(tag))
  })
}

// Tag-based permissions
function canAccess(user, entity) {
  const userRoles = user.getPlugs("Role")
  const requiredTags = entity.getPlugs("Tag")

  // Check if user has permission for any of entity's tags
  for (const tag of requiredTags) {
    const tagEntity = resolveEntity(tag)
    const requiredRole = tagEntity.getPlug("RequiredRole")

    if (requiredRole && !userRoles.includes(requiredRole)) {
      return false
    }
  }

  return true
}
```

## Tag Analytics

### Tag Metadata Enables Rich Analytics

```javascript
// Count entities by tag
function getTagStatistics() {
  const pipeline = [
    { $unwind: "$plug.Tag" },
    { $group: {
      _id: "$plug.Tag",
      count: { $sum: 1 }
    }}
  ]

  const results = db.entities.aggregate(pipeline)

  // Resolve tag UUIDs to names with metadata
  return results.map(result => {
    const tag = resolveEntity(result._id)
    return {
      name: tag.getPlug("Name"),
      color: tag.getPlug("Color"),
      icon: tag.getPlug("Icon"),
      count: result.count
    }
  })
}

// Output:
// [
//   { name: "Character", color: "#4A90E2", icon: "person", count: 247 },
//   { name: "Warrior", color: "#E94B3C", icon: "sword", count: 89 },
//   { name: "Magical", color: "#9B59B6", icon: "sparkles", count: 156 }
// ]
```

## Tag-Driven Systems

### Systems Can Query Tag Metadata

```javascript
// Rarity system uses tag metadata
class RaritySystem {
  getEntityRarity(entity) {
    const tags = entity.getPlugs("Tag").map(resolveTagEntity)

    // Check if any tag defines rarity
    for (const tag of tags) {
      if (tag.hasPlug("Rarity")) {
        return tag.getPlug("Rarity")
      }
    }

    return "Common"  // Default
  }

  applyRarityEffects(entity) {
    const rarity = this.getEntityRarity(entity)

    // Tags with GlowEffect plug add visual effects
    const tags = entity.getPlugs("Tag").map(resolveTagEntity)
    for (const tag of tags) {
      if (tag.getPlug("GlowEffect")) {
        this.addGlowEffect(entity, tag.getPlug("Color"))
      }
    }
  }
}
```

## MongoDB Storage

### Tag Entities Stored Like Any Entity

```javascript
// Tag entity in MongoDB
{
  "_id": "warrior_tag_uuid",
  "plug": {
    "Name": "Warrior",
    "Description": "Combat-focused characters",
    "Color": "#E94B3C",
    "Icon": "sword",
    "SortOrder": 5,
    "Category": "character_class_category_uuid",
    "Tag": ["character_tag_uuid"],  // Tag can have tags
    "RequiredPlugs": ["Might", "Endurance"]
  }
}

// Entity using tag
{
  "_id": "kael_uuid",
  "plug": {
    "Name": "Kael Ironwind",
    "Tag": ["warrior_tag_uuid", "character_tag_uuid"]  // UUID references
  }
}
```

### Index Tag References

```javascript
// Index for fast tag queries
db.entities.createIndex({ "plug.Tag": 1 })

// Fast tag-based queries
db.entities.find({ "plug.Tag": "warrior_tag_uuid" })  // Uses index
```

## Best Practices

### Tag Naming Conventions

```javascript
// Good - descriptive, underscore suffix
entity warrior_tag { }
entity magical_tag { }
entity boss_tag { }

// Poor - unclear naming
entity warrior { }  // Conflicts with entity names
entity tag1 { }     // Not descriptive
```

### Tag Metadata Completeness

```javascript
// Complete tag metadata
entity warrior_tag {
  plug.Name("Warrior")
  plug.Description("Combat specialists")  // Always include
  plug.Color("#E94B3C")                   // Always include
  plug.Icon("sword")                      // Always include
  plug.SortOrder(5)                       // For consistent ordering
  plug.Category("character_class")        // Group related tags
}
```

### Tag Documentation

```javascript
// Document tag hierarchies
/**
 * Tag Hierarchy:
 * - entity_type_tag
 *   - character_tag
 *     - warrior_tag
 *     - mage_tag
 *   - location_tag
 *     - dungeon_tag
 *   - item_tag
 *     - weapon_tag
 */
```

## Attribution

The tag-as-entity architecture was pioneered by **Lost Mountain Games** (2025), demonstrating that tags don't need to be simple strings—they can be rich, evolvable entities with metadata, validation rules, and behavior.

**Innovation**: Treating tags as full entities with metadata (colors, icons, descriptions) and behavior (validation rules, hierarchies), enabling rich organizational systems that evolve without schema changes.

---

**Copyright © 2025 Lost Mountain Games**
