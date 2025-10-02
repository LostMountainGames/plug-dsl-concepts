# UUID-Based Reference System

**Innovation Attribution**: The three-layer UUID reference architecture was pioneered by **Lost Mountain Games** (2025), solving entity reference stability while maintaining human-readable authoring.

## Core Innovation

Traditional game development faces a fundamental conflict:
- **Developers need**: Stable references that don't break when names change
- **Content creators need**: Human-readable names for authoring
- **Players need**: Current display names that reflect story changes

**Lost Mountain Games Innovation**: Three-layer architecture that satisfies all three needs simultaneously.

## Three-Layer Architecture

### Layer 1: Authoring (Human-Friendly)

Content creators write human-readable entity names:

```javascript
// Content creator writes this
entity Kael_Ironwind {
  plug.Name("Kael Ironwind")
  plug.Chieftain(war_chief_ironwind)      // Human-readable reference
  plug.Location(ironwolf_stronghold)      // Human-readable reference
  plug.Equipment([ironwolf_axe, clan_banner, war_horn])  // Multiple references
}
```

**Authoring Benefits**:
- Natural, readable entity references
- No need to look up UUIDs
- Fast content creation
- Easy to understand relationships

### Layer 2: Storage (Stable UUIDs)

System automatically resolves names to UUIDs for MongoDB storage:

```javascript
// What MongoDB actually stores
{
  "_id": "550e8400-e29b-41d4-a716-446655440000",
  "plug": {
    "Name": "Kael Ironwind",
    "Chieftain": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",     // UUID
    "Location": "7c9e6679-7425-40de-944b-e07fc1f90ae7",      // UUID
    "Equipment": [
      "9f8e7d6c-5b4a-3928-1716-fedcba098765",              // UUID
      "12345678-90ab-cdef-1234-567890abcdef",              // UUID
      "abcdef12-3456-7890-abcd-ef1234567890"               // UUID
    ]
  }
}
```

**Storage Benefits**:
- References never break when names change
- Globally unique identifiers
- Import/export without conflicts
- Stable database relationships

### Layer 3: Display (Current Names)

UI resolves UUIDs back to current entity names:

```javascript
// What players see in UI
{
  "Name": "Kael Ironwind",
  "Chieftain": "War Chief Ironwind",          // Resolved from UUID
  "Location": "Ironwolf Stronghold",          // Resolved from UUID
  "Equipment": [
    "Ironwolf War Axe",                       // Resolved from UUID
    "Clan Banner of the Ironwolves",          // Resolved from UUID
    "Ancient War Horn"                        // Resolved from UUID
  ]
}
```

**Display Benefits**:
- Always shows current names
- Automatic updates when entities renamed
- Player-friendly presentation
- Reflects story progression

## Name Change Propagation

### The Power of UUID References

When an entity's name changes, all references automatically update in displays:

```javascript
// Original chieftain entity
// UUID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
entity War_Chief {
  plug.Name("War Chief Ironwind")
  plug.Rank("Chieftain")
  plug.Age(55)
}

// Story progression: chieftain retires, becomes elder
entity War_Chief {
  plug.Name("Elder Ironwind")     // NAME CHANGED
  plug.Rank("Elder")
  plug.Age(56)
}

// Kael's entity in MongoDB (unchanged):
{
  "_id": "kael_uuid",
  "plug": {
    "Chieftain": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"  // UUID unchanged
  }
}

// Kael's display automatically updates:
"Chieftain: War Chief Ironwind"  →  "Chieftain: Elder Ironwind"

// NO manual updates needed
// NO broken references
// NO database migrations
```

### Everywhere-At-Once Updates

All references to renamed entity update across entire game:

```javascript
// Change one entity's name
entity Ironwolf_Axe {
  plug.Name("Bloodthirsty War Axe")  // Renamed from "Ironwolf War Axe"
}

// All references auto-update in UI:
// - Kael's equipment list
// - Armory inventory display
// - Combat log messages
// - Crafting recipe references
// - Quest reward text
// - Vendor shop displays
// - Tooltip text
// - Everywhere the axe is referenced!

// Zero manual find-replace
// Zero documentation updates
// Zero broken references
```

## UUID Generation and File Naming

### Entity Creation Flow

```javascript
// Step 1: Content creator creates entity with temporary name
// File: kael_ironwind.entity (temporary)
plug.Name("Kael Ironwind")
plug.Will(16)

// Step 2: System generates UUID on save
// UUID generated: 550e8400-e29b-41d4-a716-446655440000

// Step 3: File renamed to UUID
// File: 550e8400-e29b-41d4-a716-446655440000.entity

// Step 4: IDE shows friendly name in explorer
// IDE Display: "Kael Ironwind.entity"
// Actual File: "550e8400-e29b-41d4-a716-446655440000.entity"
```

### Benefits of UUID Filenames

```javascript
// Global uniqueness
// World A: 550e8400-e29b-41d4-a716-446655440000.entity (Kael - Warrior)
// World B: 7c9e6679-7425-40de-944b-e07fc1f90ae7.entity (Kael - Mage)
// Import both → no conflicts!

// Rename safety
// Change plug.Name("Kael") → plug.Name("Kael the Legendary")
// Filename stays: 550e8400-e29b-41d4-a716-446655440000.entity
// All references still work (UUID unchanged)

// Content sharing
// Export entity → Import to different world
// UUID prevents ID collisions
// References remain stable
```

## Reference Resolution System

### Name-to-UUID Resolution (Authoring Time)

```javascript
// Content creator writes
plug.Chieftain(war_chief_ironwind)

// System resolution process:
// 1. Parse reference: "war_chief_ironwind"
// 2. Search entity registry for matching Name
// 3. Find entity with plug.Name("War Chief Ironwind")
// 4. Extract UUID: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
// 5. Store UUID in MongoDB

// Stored value:
"Chieftain": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
```

### UUID-to-Name Resolution (Display Time)

```javascript
// UI retrieves entity
const kael = db.entities.findOne({ "_id": "kael_uuid" })
// kael.plug.Chieftain = "a1b2c3d4-e5f6-7890-abcd-ef1234567890"

// Display resolution:
// 1. Read UUID: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
// 2. Look up entity by UUID
// 3. Read plug.Name: "War Chief Ironwind"
// 4. Display: "Chieftain: War Chief Ironwind"

// If name changes to "Elder Ironwind":
// 1. Same UUID lookup
// 2. Read NEW plug.Name: "Elder Ironwind"
// 3. Display: "Chieftain: Elder Ironwind"
```

## Import/Export Safety

### The Name Collision Problem (Solved)

**Traditional Approach** (name-based references):
```javascript
// World A has: kael.entity
{
  "name": "Kael",
  "faction": "ironwolves"  // Name reference
}

// World B has: kael.entity (DIFFERENT character!)
{
  "name": "Kael",
  "faction": "mage_guild"  // Name reference
}

// Import World A → World B
// CONFLICT: Which Kael? Which faction?
// Manual resolution required
```

**UUID Approach** (unique identifiers):
```javascript
// World A: 550e8400-e29b-41d4-a716-446655440000.entity
{
  "_id": "550e8400-e29b-41d4-a716-446655440000",
  "plug": {
    "Name": "Kael",
    "Faction": "a1b2c3d4-uuid"
  }
}

// World B: 7c9e6679-7425-40de-944b-e07fc1f90ae7.entity
{
  "_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "plug": {
    "Name": "Kael",
    "Faction": "b2c3d4e5-uuid"
  }
}

// Import World A → World B
// No conflict! Different UUIDs = different entities
// Both "Kael" entities coexist safely
```

### Content Package Distribution

```javascript
// Create content package
const package = {
  entities: [
    "550e8400-uuid.entity",  // Kael
    "a1b2c3d4-uuid.entity",  // Iron Wolves faction
    "9f8e7d6c-uuid.entity"   // Ironwolf Axe
  ],
  references: {
    "kael → faction": "a1b2c3d4-uuid",
    "kael → weapon": "9f8e7d6c-uuid"
  }
}

// Import to any world
// UUIDs guarantee no conflicts
// References remain intact
// Works first time, every time
```

## LLM Integration Benefits

### AI-Friendly Reference System

Large Language Models can work with either names or UUIDs:

```javascript
// LLM generates entity with name-based references
entity New_Character {
  plug.Name("Mysterious Stranger")
  plug.Location(tavern_location)      // Name reference - LLM friendly
  plug.Ally(kael_ironwind)            // Name reference - LLM friendly
}

// System resolves to UUIDs automatically
{
  "_id": "generated_uuid",
  "plug": {
    "Name": "Mysterious Stranger",
    "Location": "tavern_uuid",        // Auto-resolved
    "Ally": "kael_uuid"               // Auto-resolved
  }
}

// LLM can also use UUIDs explicitly if needed
plug.Location("7c9e6679-7425-40de-944b-e07fc1f90ae7")  // Direct UUID
```

### AI Content Generation Workflow

```javascript
// 1. AI generates entity with human-readable references
const aiGenerated = `
entity Shadow_Assassin {
  plug.Name("Shadow Assassin")
  plug.Target(kael_ironwind)
  plug.HiredBy(evil_mage)
  plug.Location(dark_alley)
}
`

// 2. System validates references exist
// 3. System resolves names to UUIDs
// 4. System stores with stable references
// 5. AI can query result using names or UUIDs

// AI workflow never sees UUIDs unless it wants to
// But benefits from UUID stability
```

## Performance Optimization

### UUID Resolution Caching

```javascript
// Cache frequently resolved UUIDs
const resolutionCache = new Map()

function resolveUUID(uuid) {
  // Check cache first
  if (resolutionCache.has(uuid)) {
    return resolutionCache.get(uuid)
  }

  // Look up entity
  const entity = db.entities.findOne({ "_id": uuid })
  const name = entity.plug.Name

  // Cache result
  resolutionCache.set(uuid, name)

  return name
}

// Sub-millisecond resolution for cached lookups
```

### Batch Resolution

```javascript
// Resolve multiple UUIDs at once
function resolveBatch(uuids) {
  const uncached = uuids.filter(uuid => !resolutionCache.has(uuid))

  if (uncached.length > 0) {
    // Single DB query for all uncached UUIDs
    const entities = db.entities.find({
      "_id": { $in: uncached }
    })

    // Cache results
    entities.forEach(entity => {
      resolutionCache.set(entity._id, entity.plug.Name)
    })
  }

  // Return all names from cache
  return uuids.map(uuid => resolutionCache.get(uuid))
}
```

## Best Practices

### Reference Validation

```javascript
// Validate references at authoring time
function validateReference(refName) {
  const entity = entityRegistry.findByName(refName)

  if (!entity) {
    console.error(`Reference not found: ${refName}`)
    console.log(`Did you mean: ${suggestSimilar(refName)}?`)
    return null
  }

  return entity.uuid
}

// Help content creators catch typos early
```

### Reference Documentation

```javascript
// Document entity relationships
entity Kael {
  plug.Name("Kael Ironwind")

  // References (auto-documented)
  plug.Chieftain(war_chief_ironwind)    // → War Chief Ironwind
  plug.Location(ironwolf_stronghold)    // → Ironwolf Stronghold
  plug.Equipment([ironwolf_axe])        // → Ironwolf War Axe

  // IDE can show resolved names as hover tooltips
  // Content creator sees: plug.Chieftain(war_chief_ironwind)
  // IDE tooltip shows: "War Chief Ironwind (UUID: a1b2c3d4...)"
}
```

## Attribution

The three-layer UUID reference system was pioneered by **Lost Mountain Games** (2025), solving the conflict between stable storage, human-readable authoring, and dynamic name displays.

**Innovation**: Separating authoring (names), storage (UUIDs), and display (resolved names) into three distinct layers that work together seamlessly.

---

**Copyright © 2025 Lost Mountain Games**
