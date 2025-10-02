# No Classes, Just Plugs

**Innovation Attribution**: The "No Classes, Just Plugs" architecture was pioneered by **Lost Mountain Games** (2025) and represents a fundamental reimagining of entity systems in game development.

## Core Innovation

Traditional game engines define entity types through class hierarchies:
```
Character extends Entity
  Warrior extends Character
  Mage extends Character
Location extends Entity
  Dungeon extends Location
Item extends Entity
  Weapon extends Item
```

**Lost Mountain Games Innovation**: Eliminate classes entirely. Entities are simply collections of plugs.

## Architecture Foundation

### Entities Are Plug Collections

**Principle**: An entity is nothing more than a collection of plugs stored in MongoDB. No class definitions, no inheritance hierarchies, no predefined types.

```javascript
// This is a "character" because it has character-like plugs
{
  "_id": "kael_uuid",
  "plug": {
    "Name": "Kael Ironwind",
    "Will": 16,
    "Might": 18,
    "Personality": "Gruff but loyal"
  }
}

// This is a "location" because it has location-like plugs
{
  "_id": "grotto_uuid",
  "plug": {
    "Name": "Whispering Grotto",
    "LocationType": "AncientRuin",
    "Region": "NorthernWildlands"
  }
}

// This is BOTH because it has BOTH types of plugs
{
  "_id": "worldtree_uuid",
  "plug": {
    "Name": "Heart of the Worldtree",
    "LocationType": "CosmicNexus",
    "Will": 20,
    "Personality": "Ancient, wise, speaks in riddles",
    "CanSpeak": true
  }
}
```

**Revolutionary Insight**: The system doesn't enforce categories. MongoDB stores whatever plugs you define. Systems query for plugs they care about.

### No Type Discrimination

**Traditional Class-Based**:
```javascript
// Must check entity type
if (entity instanceof Character) {
  processCharacter(entity)
} else if (entity instanceof Location) {
  processLocation(entity)
}

// Talking location? Doesn't fit anywhere!
```

**Pluggable Architecture**:
```javascript
// Check for plugs, not types
if (entity.hasPlug("Will") && entity.hasPlug("Personality")) {
  processSentientBeing(entity)
}

if (entity.hasPlug("LocationType") && entity.hasPlug("Region")) {
  processLocation(entity)
}

// Talking location works perfectly!
// It has both sets of plugs
```

## MongoDB Storage Pattern

### Pure Document Storage

Every entity is stored as a MongoDB document with plugs as fields:

```javascript
// Character entity
{
  "_id": "550e8400-e29b-41d4-a716-446655440000",
  "plug": {
    "Name": "Kael Ironwind",
    "Age": 34,
    "Clan": "a1b2c3d4-uuid",
    "Will": 12,
    "Might": 16,
    "Endurance": 14,
    "Speed": 10
  }
}

// Item entity
{
  "_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "plug": {
    "Name": "Ironwolf War Axe",
    "ItemType": "Weapon",
    "Damage": "2d6+2",
    "Weight": 3.5
  }
}

// Sentient item entity (item + character plugs!)
{
  "_id": "9f8e7d6c-5b4a-3928-1716-fedcba098765",
  "plug": {
    "Name": "Soulrend Glaive",
    "ItemType": "MagicalGlaive",
    "Damage": "3d6+3",
    "Will": 15,
    "Personality": "Thirsts for battle",
    "Memories": "Recalls every soul claimed"
  }
}
```

**Key Point**: MongoDB doesn't care that the third entity mixes "item" and "character" plugs. It just stores the document.

### Query by Plugs

Systems find entities by querying for required plugs:

```javascript
// Find all entities with combat capability
db.entities.find({
  "plug.Damage": { $exists: true }
})
// Returns: weapons, characters with unarmed attacks,
//          sentient weapons, combat-capable locations

// Find all sentient entities
db.entities.find({
  "plug.Will": { $exists: true },
  "plug.Personality": { $exists: true }
})
// Returns: characters, sentient items, sentient locations,
//          talking animals, conscious AI

// Find all things that can move
db.entities.find({
  "plug.Speed": { $exists: true }
})
// Returns: characters, vehicles, mobile fortresses,
//          animated objects, flying castles
```

## System Design Patterns

### Plug-Based Systems

Systems operate on entities based on plug presence:

```javascript
// Combat System - works with ANY entity that has combat plugs
class CombatSystem {
  canAttack(entity) {
    return entity.hasPlug("Damage")
  }

  attack(attacker, defender) {
    if (!this.canAttack(attacker)) return

    const damage = attacker.getPlug("Damage")
    // Process combat...
  }
}

// Works with:
// - Characters (have Damage plug)
// - Sentient weapons (have Damage plug)
// - Combat locations like "The Moving Castle" (has Damage plug)
// - Magical traps (have Damage plug)
// - Anything with a Damage plug!
```

### No Special Cases

**Traditional Class-Based** (brittle):
```javascript
// Special handling for each entity type
class CombatSystem {
  attack(attacker, defender) {
    if (attacker instanceof Character) {
      return this.characterAttack(attacker, defender)
    } else if (attacker instanceof Weapon) {
      throw new Error("Weapons can't attack directly!")
    } else if (attacker instanceof Location) {
      throw new Error("Locations can't attack!")
    }
    // Special cases multiply endlessly...
  }
}
```

**Pluggable Architecture** (flexible):
```javascript
// One rule: if it has Damage plug, it can attack
class CombatSystem {
  attack(attacker, defender) {
    if (!attacker.hasPlug("Damage")) return

    const damage = attacker.getPlug("Damage")
    this.dealDamage(defender, damage)
  }
}

// Talking sword attacks? Has Damage plug. Works.
// Walking castle attacks? Has Damage plug. Works.
// Living spell attacks? Has Damage plug. Works.
// No special cases needed!
```

## Benefits

### 1. Unlimited Creativity

No artificial boundaries prevent creative entity design:

```javascript
// Want a location that's also a boss enemy?
entity The_Moving_Castle {
  // Location plugs
  plug.LocationType("MobileFortress")
  plug.Region("NorthernWildlands")
  plug.Contains([throne_room, crystal_core])

  // Boss enemy plugs
  plug.Will(18)
  plug.Might(20)
  plug.Damage("5d6+10")
  plug.BossLevel(15)

  // System doesn't care - just stores it
}
```

### 2. No Refactoring for New Features

Add plugs without touching existing code:

```javascript
// Week 1: Basic character
entity Kael {
  plug.Name("Kael Ironwind")
  plug.Will(16)
}

// Week 10: Add magic system
entity Kael {
  plug.Name("Kael Ironwind")
  plug.Will(16)
  plug.MagicPower(25)  // New plug - no code changes
}

// Week 20: Add reputation
entity Kael {
  plug.Name("Kael Ironwind")
  plug.Will(16)
  plug.MagicPower(25)
  plug.Reputation(iron_wolves, "Revered")  // Another new plug
}

// Entity class doesn't exist to modify
// MongoDB stores whatever plugs you add
```

### 3. Cross-Game System Support

Different game systems can coexist:

```javascript
// Reavers Universe character
entity Kael_Reavers {
  plug.Name("Kael")
  plug.Will(16)
  plug.Might(18)
}

// D&D 5e character (same entity, different plugs!)
entity Kael_DnD {
  plug.Name("Kael")
  dnd.Strength(18)
  dnd.Dexterity(14)
  dnd.Class("Fighter")
}

// Hybrid character (why not?)
entity Kael_Hybrid {
  plug.Name("Kael")
  plug.Will(16)        // Reavers
  dnd.Strength(18)     // D&D
  custom.Luck(12)      // Custom system
  // No class hierarchy to prevent this
}
```

## Comparison to Traditional ECS

### Traditional Entity-Component Systems

ECS frameworks still define component types:

```javascript
// Component definitions required
class HealthComponent {
  health: number
  maxHealth: number
}

class PositionComponent {
  x: number
  y: number
}

// Entity has component instances
entity.addComponent(new HealthComponent())
entity.addComponent(new PositionComponent())
```

### Pluggable Architecture

No component classes—just plugs:

```javascript
// No component definitions needed
entity Warrior {
  plug.Health(100)
  plug.MaxHealth(100)
  plug.Position({ x: 10, y: 20 })
  // Just data, stored in MongoDB
}
```

**Advantage**: Add new "component types" (plugs) without defining classes or schemas.

## MongoDB Atlas Enabler

This architecture is only possible because of MongoDB's schemaless nature:

**Relational Database** (impossible):
```sql
-- Must define all possible columns
CREATE TABLE entities (
  id UUID PRIMARY KEY,
  name VARCHAR(100),
  health INT,
  -- Every possible plug needs a column!
  -- Can't add new plugs without ALTER TABLE
);
```

**MongoDB Atlas** (enables plug architecture):
```javascript
// Store whatever plugs you want
{
  "_id": "uuid",
  "plug": {
    "Name": "...",
    "Health": 100,
    // Add any plug, anytime
    "NewPlugAddedToday": "value"
  }
}
```

## Best Practices

### Plug Naming Consistency

Use consistent naming for similar concepts:

```javascript
// Good - consistent naming
plug.MaxHealth(100)
plug.MaxMana(100)
plug.MaxStamina(100)

// Poor - inconsistent
plug.MaxHealth(100)
plug.ManaLimit(100)
plug.StaminaCap(100)
```

### Plug Discovery

Document what plugs exist:

```javascript
// Create plug registry documentation
const CommonPlugs = {
  Identity: ["Name", "Description", "Age"],
  Combat: ["Damage", "Health", "Armor"],
  Spatial: ["Location", "Position", "Region"],
  Mental: ["Will", "Personality", "Memories"]
}

// But don't enforce it - just document it
```

### System Documentation

Document what plugs systems expect:

```javascript
/**
 * CombatSystem processes entities with combat capability
 * Required plugs: Damage
 * Optional plugs: Armor, CriticalChance
 */
class CombatSystem {
  // ...
}
```

## Attribution

The "No Classes, Just Plugs" architecture was pioneered by **Lost Mountain Games** (2025), enabling unlimited entity composition by eliminating rigid class hierarchies in favor of pure plug-based entity definition.

**Innovation**: Entities as pure plug collections stored in MongoDB, with no class definitions or type constraints.

---

**Copyright © 2025 Lost Mountain Games**
